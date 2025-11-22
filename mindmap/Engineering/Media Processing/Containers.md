[[Codec]]
When a camera records, it produces **raw streams** of bits:
- One for **video** (frames compressed by a video codec like H.264)
- One for **audio** (samples compressed by an audio codec like AAC)
- Optional ones for **subtitles, captions, metadata, thumbnails, timestamps, etc.**

If we just dumped these into a single file, the player wouldn’t know:
- Which bytes belong to which track
- When each audio sample should play relative to each video frame
- Where the file starts/ends, or how to seek inside it

The **container format** solves all that: it’s a _file structure and index_ that wraps those separate coded streams together in an organized way.

```
[Container: MP4 file]
 ├── Track #1 (video)
 │     ├── Frame #1 (H.264 encoded)
 │     ├── Frame #2 ...
 │     └── ...
 ├── Track #2 (audio)
 │     ├── Chunk #1 (AAC encoded)
 │     ├── Chunk #2 ...
 │     └── ...
 ├── Metadata (duration, codec info)
 └── Index (byte offsets, timestamps)

```
## What happens when a player opens a container

When a video player loads a file like `movie.mp4` or `clip.webm`, it:
1. **Reads the container header** – learns which tracks exist, which codecs they use, duration, resolution, etc.
2. **Builds a timeline** – using timestamps stored inside the container.
3. **Requests the right bytes** – if it’s a stream, the player fetches chunks corresponding to time ranges.
4. **Hands off data** – to the appropriate decoders (video → H.264 decoder, audio → AAC decoder).
5. **Synchronizes** – using timestamps so lip-sync stays perfect.

The container doesn’t decode anything itself — it’s just an index and wrapper so decoding works correctly.

Deep dive in the structure of Mp4 and WebM containers
- [[MP4 Container]]
- [[WebM Container]]