An MP4 file is a **binary container** made up of small pieces called **boxes** (also known as _atoms_).  
Each box has:

- a **size field** (4 bytes)
- a **type field** (4 bytes — usually ASCII letters like `ftyp`, `moov`, `mdat`, etc.)
- and then **data** that follows (the payload).
Every single thing in MP4, from the top-level file to nested track tables, follows this same simple pattern.

```
[ size ][ type ][ data ... ]
```
So the file looks like a long sequence of boxes — some of which contain _other boxes_.

# The top-level structure

A simple MP4 file might look like this:
```
+----------------+
| ftyp | moov | mdat |
+----------------+
```
### - `ftyp` (File Type Box)
- First thing in the file (the “signature”).
- Declares which _brand_ (variant) of MP4 this is.
- Example: `isom`, `mp42`, `avc1`, `dash`.

### - `moov` (Movie Box)
- Contains all metadata (tracks, timing, codec info).
- Think of it like the “table of contents”.
- Inside `moov`, there are nested boxes like:
    - `mvhd` — movie header (overall duration, timescale)
    - `trak` — one per media track (video, audio)
        - `tkhd` — track header (width, height)
        - `mdia` — media info
            - `mdhd` (media header: timebase)
            - `hdlr` (handler: says “vide” or “soun”)
            - `minf`
                - `stbl` (sample table)
                    - `stts` (timestamp table)
                    - `stsz` (sample sizes)
                    - `stco` (chunk offsets)
                    - `stsc` (sample-to-chunk map)

So `moov` tells you _how to interpret the bytes_ in the next section.

### - `mdat` (Media Data Box)
- This holds the **actual encoded audio and video frames** (as bytes).
- The tables inside `moov` point into this region by byte offset.

# How the player reads the file

When you open an MP4:
1. **Read `ftyp`** – checks if it’s compatible.
2. **Read `moov`** – builds index of frames (offsets, timestamps).
3. **Go to `mdat`** – fetches frame #1, #2, etc., according to the tables.
4. **Send to decoders** – each frame’s codec ID (`avc1`, `hev1`, etc.) tells which decoder to use.

# Fragmented MP4 (fMP4)

For streaming, MP4 is often split into an **initialization segment** and many small **media fragments**.
### Init segment:
Contains only
- `ftyp`
- `moov` (metadata, codecs, track info)
### Media fragments:
Contain:
- `moof` (movie fragment header)
- `mdat` (actual video/audio bytes for a few seconds)

Each fragment has its own timing info in `moof`.  
This allows you to stream piece by piece without rewriting the entire `moov` each time.