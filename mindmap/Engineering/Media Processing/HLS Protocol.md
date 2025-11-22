[[Codec]] [[Containers]]
- Its stands for HTTP Live Streaming.
- t's an HHTP based protocol that breaks media into small segments and uses playlists to manage delivery and adaption
## Terminology
#### Playlist/Manifest
- It uses a playlist file in the m3u8 forat. It essentially lists all available segments for a video playback. 
- There are two major playlist type -> master playlist (listing of various streams - different bitrates and resolutions) so player can choose dynamically, and media playlist, which lists sequential media segments
#### Media Segments
Actual video/audio content broken into small files (segments) of 2-10s that the player can download one by one. 
#### Adaptive Bitrate
The player monitors the network conditions and switches between variant streams (in master playlist) to deliver optimum quality with minimal buffering
#### Live vs VOD
- VOD (on demand): All segments are present, playlist includes full list and ends with `#EXT-X-ENDLIST`
- Live Stream: segments are produced in real time, playlist is continuously updated and player keeps fetching the next chunk.
## Detailed Workflow
### Step 1: Encoding & multiple renditions
- Source video is encoded into multiple **bitrates/resolutions** (e.g., 1080p@5Mbps, 720p@3Mbps, 480p@1Mbps).
- Audio also encoded (e.g., AAC).
- The codecs used must be compatible with target devices (H.264 + AAC is most universal).
### Step 2: Segmenting / packaging
- Encoded streams are divided into small media segments (2-10s long). For example: `seg_0001.ts`, `seg_0002.ts`, … or `seg_0001.m4s`, `seg_0002.m4s` (for fMP4).
- A **media playlist** is generated for each variant (bitrate). Example playlist lines:
```
#EXTM3U
#EXT-X-TARGETDURATION:4
#EXTINF:4.0,
seg_0001.ts
#EXTINF:4.0,
seg_0002.ts
…

```
### Step 3: Master playlist
- A master playlist lists variant media playlists.
```
#EXTM3U
#EXT-X-STREAM-INF:BANDWIDTH=5000000,RESOLUTION=1920x1080
hq/playlist.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=3000000,RESOLUTION=1280x720
med/playlist.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=1000000,RESOLUTION=854x480
low/playlist.m3u8
```
### Step 4: CDN/HTTP delivery
- Segments and playlists are hosted on HTTP servers or CDNs (Content Delivery Networks). Because it’s just HTTP, it scales easily and passes through firewalls.
- A player uses HTTP to fetch the playlist then segments.
### Step 5: Playback by the client
- The client downloads the master playlist → chooses a variant (initially may pick a lower bitrate).
- It then fetches the media playlist → downloads segments in order and buffers a few seconds.
- The ABR logic monitors network/CPU and may switch variants mid-stream (when next segment is requested) for smoother playback.
### Step 6: Live mode behaviour
- For live streams, the media playlist is **sliding window**: it only includes the most recent segments (e.g., last 3-6 segments).
- The client periodically reloads the playlist (every few seconds) to discover new segments and continues playback.
### Step 7: Latency, start time & end

- The live stream’s latency depends on segment length + buffer + player fetch delays.
- For VOD, playlist ends with `#EXT-X-ENDLIST`.
- For live, there is no end list and playback continues as segments arrive.