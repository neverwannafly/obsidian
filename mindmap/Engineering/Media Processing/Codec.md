- Simply put, codec is a technology and decompresses and compresses digital media like audio and video
- It compresses RAW audio/video into smaller bitstreams and decompresses them on playback
- Video codecs -> H.264/AVC, H.265/HEVC, VP8/VP9, AV1
- Audio codecs -> AAC, Opus, MP3, AC-3
Without codecs, streaming would be impossible — raw 1080p video at 60 fps would require hundreds of megabits per second.

## Major Modern Codecs — A Timeline

| Year  | Codec                 | Organization                                            | Typical Container | Key Features / Notes                                                           |
| ----- | --------------------- | ------------------------------------------------------- | ----------------- | ------------------------------------------------------------------------------ |
| ~2003 | **H.264 / AVC**       | ITU-T & ISO (MPEG-4 Part 10)                            | MP4, TS           | Foundation of streaming era; very efficient, hardware-decoded everywhere.      |
| 2013  | **H.265 / HEVC**      | ITU-T & ISO (MPEG-H Part 2)                             | MP4, TS           | ~50% smaller bitrates than H.264; 4K support; patent-heavy licensing.          |
| 2013  | **VP9**               | Google / open source                                    | WebM, MKV         | Royalty-free rival to H.265; widely used by YouTube; great for web.            |
| 2018  | **AV1**               | AOM Alliance (Google, Netflix, Amazon, Microsoft, etc.) | MP4, WebM         | ~30% smaller than VP9/H.265; royalty-free; CPU-heavy, hardware support rising. |
| 2020+ | **VVC (H.266)**       | MPEG / Fraunhofer                                       | MP4               | ~30–50% smaller again vs H.265, targeted for 8K/HDR, licensing complex.        |
| 2023+ | **EVC / LCEVC / AV2** | Various                                                 | MP4               | Experimental next-gen codecs with different focuses (e.g. enhancement layers). |

## Compression Efficiency Comparison

Approximate bitrate needed for equal quality (lower = better):

| Codec        | Relative Size (vs H.264 = 100%) | Typical Use                        |
| ------------ | ------------------------------- | ---------------------------------- |
| H.264 / AVC  | 100 %                           | Legacy, universal hardware support |
| H.265 / HEVC | ~50 – 60 %                      | 4K UHD, broadcast, OTT             |
| VP9          | ~55 – 65 %                      | YouTube, web browsers              |
| AV1          | ~35 – 45 %                      | Netflix, YouTube, modern browsers  |
| VVC          | ~25 – 35 %                      | Early 8K adoption, future standard |
## Codec vs Bitrate vs Quality
- **Bitrate (kbps)** = how much data per second.
- **Resolution (1080p, 4K)** = number of pixels.
- **Codec efficiency** determines how low you can push bitrate for the same visual fidelity.

Rough example for 1080p @ 30 fps.

|Codec|Bitrate for “Good” Quality|
|---|---|
|H.264|4–5 Mbps|
|H.265 / VP9|2.5–3 Mbps|
|AV1|1.8–2.5 Mbps|
# Practical Working of Codecs
### Step 1. Frame prediction (temporal compression)
- Instead of storing every frame completely, codecs store a **reference frame** (an _I-frame_) and then, for the next frames, store **only the changes**.
- These “change frames” come in two flavors:
    - **P-frames (predictive)** – reference **past** frames.
    - **B-frames (bidirectional)** – reference **past and future** frames for even better prediction.
- This is why you can’t instantly jump to any frame in a compressed stream—you may need to decode back to the last I-frame first.

### **Step 2. Block division (spatial segmentation)**
- Each frame is split into small rectangular **blocks** of pixels.
    - In H.264 they’re called **macroblocks** (16×16).
    - In H.265/HEVC they’re **Coding Tree Units (CTUs)** that can vary in size (up to 64×64).
- This lets the codec handle different parts of the image separately—static sky vs. moving person

### **Step 3. Motion estimation and compensation**
- The encoder searches the previous frames for where each block **moved**.
- Instead of re-sending pixels, it sends a **motion vector** like “this 16×16 block shifted 5 pixels right and 2 up.”
- At decode time, the player uses those vectors to reconstruct the new frame.

### **Step 4. Transform + Quantization (frequency compression)**
After prediction there’s still a small _difference_ (residual) between predicted and real pixels.
- The codec runs a **Discrete Cosine Transform (DCT)**-like math on each block to turn pixel values into **frequency coefficients** (how much smooth color vs. fine detail).
- Then it **quantizes** them: throws away small, high-frequency values that the eye won’t notice—this is where most data reduction happens.
    - Higher quantization → smaller file, more blur.

### **Step 5 Entropy coding (lossless bit-packing)**
Now we have a bunch of numbers (motion vectors + quantized coefficients). The codec compresses them further with statistical coding:
- **Huffman** or **Arithmetic/CABAC** codes shorter symbols for frequent values.
- No quality loss here—just squeezing out redundant bits.