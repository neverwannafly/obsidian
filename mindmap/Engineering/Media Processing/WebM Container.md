WebM is built on **EBML** — the **Extensible Binary Meta Language**.  
EBML is like a binary version of XML:
- Every element has a **tag ID**, a **length**, and a **value**.
- Elements can contain other elements (nested structure).
- It’s self-describing — a parser can skip anything it doesn’t understand.

So where MP4 uses `[size][type][data]`, WebM uses `[elementID][length][data]`.
A simple WebM file looks like this:

```
[EBML]
[Segment]
   ├── [Info]
   ├── [Tracks]
   ├── [Cluster]  (1)
   ├── [Cluster]  (2)
   ├── [Cues]
   └── [Tags]

```
Each **Cluster** contains a short section of media data (like a few seconds of video/audio).  
`Info` and `Tracks` describe what’s inside; `Cues` act like an index for seeking.

A typical webM file will have the following structure
```xml
<EBML/>  <!-- file header declaring “webm” -->

<Segment>                       <!-- the root container (Matroska term) -->
  <Info>                        <!-- global timing/duration -->
    <TimecodeScale>1000000</TimecodeScale>  <!-- 1 time unit = 1 ms (typical) -->
    <Duration>…</Duration> <!-- This is optional in live playback -->
  </Info>

  <Tracks>                      <!-- list of streams -->
    <TrackEntry number="1" type="video"><CodecID>V_VP9</CodecID> …</TrackEntry>
    <TrackEntry number="2" type="audio"><CodecID>A_OPUS</CodecID> …</TrackEntry>
  </Tracks>

  <!-- Repeats along the timeline: “slices” of actual media -->
  <Cluster timecode="0">
    <SimpleBlock track="1" keyframe="1">…video frame…</SimpleBlock>
    <SimpleBlock track="2">…audio packet…</SimpleBlock>
    …
  </Cluster>

  <Cluster timecode="2000"> … </Cluster>   <!-- base time in ms here -->
  …

  <Cues/>   <!-- index for fast seeking (optional for live) -->
  <Tags/>   <!-- metadata (optional) -->
</Segment>

```
## How MediaRecorder produces WebM chunks?
When we do `mediaRecorder.start(timeslice)`, it emits **Blobs** periodically:
- **Chunk #1 (the first Blob)**: contains the **EBML header + Segment + Info + Tracks**, then one or more **Clusters** with the first few seconds of media.  This acts like an **initialization segment + first media segment**.
- **Chunk #N (subsequent Blobs)**: typically contain **only more Clusters** (no EBML/Tracks repeated). By spec, **individual Blobs do not have to be independently playable**; only the **concatenation** of all Blobs from the recording must be a valid file.
```
# Good reconstruction order

[Chunk1: EBML + Segment + Info + Tracks + Cluster A]
[Chunk2: Cluster B]
[Chunk3: Cluster C]
...
→ Concatenate: Header (once) + A + B + C + …

```

## Practical Implications dealing with webM
#### Chunk concatenation
```
cat ~/Desktop/hhiq2ywf_1763553066604_1763553076655.webm ~/Desktop/hhiq2ywf_1763553076655_1763553086686.webm > new.webm

```
this generates a full contained webm file which is playable.
More information can be found on this thread - 
https://groups.google.com/a/webmproject.org/g/webm-discuss/c/6ySds58ZhEQ

#### Cluster Generation
- Usually when MediaRecorder outputs webM segments, subsequent chunks can start from middle of cluster. It's not important that a chunk will always end when a cluster ends. 
- Example, chunk 2 can contain half of cluster 3 and chunk 3 can just start directly from remaining part of cluster 3. 

#### How to play arbitrary webm segments?
##### 1. Extract header information from the 1st webM chunk

```python
import sys
import argparse
from pathlib import Path

EBML_ID      = b"\x1A\x45\xDF\xA3"
SEGMENT_ID   = b"\x18\x53\x80\x67"
CLUSTER_ID   = b"\x1F\x43\xB6\x75"

def find(b: bytes, pat: bytes, start=0) -> int:
    return b.find(pat, start)

def extract_header(infile: Path, out_path: Path):
    data = infile.read_bytes()

    ebml_off = find(data, EBML_ID, 0)
    if ebml_off == -1:
        raise RuntimeError("EBML header not found")

    seg_off = find(data, SEGMENT_ID, ebml_off)
    if seg_off == -1:
        raise RuntimeError("Segment element not found")

    first_cluster_off = find(data, CLUSTER_ID, seg_off)
    if first_cluster_off == -1:
        raise RuntimeError("No Cluster found (file may have no media)")

    header_bytes = data[:first_cluster_off]
    out_path.write_bytes(header_bytes)
    print(f"[✓] Header extracted → {out_path} ({len(header_bytes)} bytes)")

def main():
    ap = argparse.ArgumentParser(description="Extract WebM header and complete Clusters")
    ap.add_argument("--header", metavar="INPUT.webm",
                    help="Extract header (EBML+Segment+Info+Tracks…) to header.raw")
    ap.add_argument("--clusters", metavar="OUTPUT.raw",
                    help="Write all complete Cluster elements from given files")
    ap.add_argument("inputs", nargs="*", help="Input WebM chunk files (for --clusters)")
    args = ap.parse_args()

    if args.header:
        extract_header(Path(args.header), Path("header.raw"))
    if args.clusters:
        if not args.inputs:
            print("Provide one or more input files when using --clusters")
            sys.exit(1)
        extract_clusters(Path(args.clusters), args.inputs)
    if not args.header and not args.clusters:
        ap.print_help()

if __name__ == "__main__":
    main()
```

##### 2. Extract Clusters
- A cluster is created across many segments. Its not necessary that wherever MediaRecorder chunks a segment, we'll get a cluster end there only. 
- A cluster usually spans across multiple webm segments. We can extract one cluster from usually 3-4 webm segments using the following code - 
```python
#!/usr/bin/env python3
"""
extract_clusters_concat.py

Concatenates multiple WebM chunks in memory and extracts complete Cluster elements.
Skips incomplete/truncated Clusters.

Usage:
  python extract_clusters_concat.py --out clusters.raw chunk1.webm chunk2.webm chunk3.webm
"""

from pathlib import Path
import argparse

CLUSTER_ID = b"\x1F\x43\xB6\x75"  # Cluster element ID

def read_vint_size_at(buf: bytes, pos: int):
    """Read EBML VINT size from position `pos` in buffer.
       Returns (value, length, is_unknown) or (None, 0, False) if invalid."""
    if pos >= len(buf):
        return (None, 0, False)
    first = buf[pos]
    mask = 0x80
    size_len = 1
    while size_len <= 8 and not (first & mask):
        mask >>= 1
        size_len += 1
    if size_len > 8 or pos + size_len > len(buf):
        return (None, 0, False)

    # Build value with leading length bit cleared
    value = first & (~mask)
    for i in range(1, size_len):
        value = (value << 8) | buf[pos + i]

    # Unknown size check (all-ones)
    all_ones = (1 << (7 * size_len)) - 1
    is_unknown = (value == all_ones)
    return (value, size_len, is_unknown)

def extract_clusters_concat(out_path: Path, inputs):
    """Concatenate multiple .webm chunks and extract all complete clusters."""
    # Read all input files sequentially
    combined = b"".join(Path(p).read_bytes() for p in inputs)
    total_len = len(combined)
    print(f"[i] Combined length = {total_len:,} bytes from {len(inputs)} file(s)")

    pos = 0
    total_clusters = 0

    with open(out_path, "wb") as out:
        while True:
            idx = combined.find(CLUSTER_ID, pos)
            if idx == -1:
                break

            size_pos = idx + len(CLUSTER_ID)
            val, size_len, is_unknown = read_vint_size_at(combined, size_pos)
            if val is None:
                break

            if not is_unknown:
                end_pos = idx + len(CLUSTER_ID) + size_len + val
                if end_pos <= total_len:
                    out.write(combined[idx:end_pos])
                    total_clusters += 1
                    pos = end_pos
                else:
                    # incomplete cluster at end
                    break
            else:
                # unknown-size cluster — bound by next Cluster or EOF
                next_cluster = combined.find(CLUSTER_ID, idx + len(CLUSTER_ID))
                if next_cluster != -1:
                    out.write(combined[idx:next_cluster])
                    total_clusters += 1
                    pos = next_cluster
                else:
                    # unknown size till EOF: treat as incomplete, skip
                    break

    print(f"[✓] Extracted {total_clusters} complete clusters → {out_path}")

def main():
    ap = argparse.ArgumentParser(description="Concatenate WebM chunks & extract complete clusters")
    ap.add_argument("--out", required=True, help="Output file for clusters (e.g. clusters.raw)")
    ap.add_argument("inputs", nargs="+", help="Input .webm chunks in order")
    args = ap.parse_args()
    extract_clusters_concat(Path(args.out), args.inputs)

if __name__ == "__main__":
    main()

```

##### 3. Do simple concatenation operation
Now, we can simply concatenate the extracted header file with the cluster and get a playable video file. 
