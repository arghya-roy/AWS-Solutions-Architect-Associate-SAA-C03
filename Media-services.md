# AWS Exam Study Guide: Media Services
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** Amazon Elastic Transcoder · Amazon Kinesis Video Streams

---

## Media Services at a Glance

| Service | What It Does | Direction | Latency | Exam Trigger Words |
|---|---|---|---|---|
| **Elastic Transcoder** | Convert/transcode stored media files to different formats | Stored (VOD) | Batch (minutes) | "convert video format", "MP4 to HLS", "video on demand", "transcode" |
| **Kinesis Video Streams** | Ingest, store, and process live video streams from devices | Live streaming | Real-time / near-real-time | "live video", "IP camera", "WebRTC", "real-time video analytics", "dashcam" |

---

---

## 1. Amazon Elastic Transcoder

### What Problem Does It Solve?

Video files come in dozens of formats (MP4, MOV, AVI, MKV), codecs (H.264, HEVC, VP9), and resolutions (4K, 1080p, 720p, 480p). Different devices — iPhones, Android phones, Smart TVs, web browsers — require different formats and bitrates. Without transcoding, a 4K MOV file shot on a camera cannot be streamed on a mobile phone over a slow connection.

**Elastic Transcoder solves this by:**
- Taking an input video file stored in **Amazon S3**
- Converting it to one or more output formats optimized for different devices
- Storing the output files back in **Amazon S3**
- All without managing any servers or encoding infrastructure

**Real-World Example:**
> A media company uploads raw TV episodes (4K ProRes files, ~50GB each) to S3 after filming. Elastic Transcoder automatically converts each episode into:
> - 1080p H.264 MP4 for Smart TVs
> - 720p HLS for iOS/macOS streaming
> - 480p DASH for Android streaming
> - 360p WebM for older browsers
>
> All four outputs are stored in S3 and served via CloudFront to global viewers.

---

### Core Concepts

#### Pipeline
- A **queue** that processes transcoding jobs.
- Has an input S3 bucket and one or more output S3 buckets.
- Controls throughput — multiple pipelines can run in parallel.
- **Status**: Active or Paused.

> **Analogy:** A pipeline is like a factory assembly line. You feed raw video in one end; finished videos come out the other.

#### Job
- A single transcoding request.
- Specifies: input file (S3 key), output preset(s), output S3 location, thumbnail settings.
- One job can produce **multiple outputs** simultaneously (e.g., 1080p + 720p + 480p in one job).
- Jobs are submitted to a pipeline and processed in order.

#### Preset
- A **template** defining the output format, codec, bitrate, resolution, and audio settings.
- Two types:
  - **System Presets**: Pre-built by AWS (e.g., "Generic 1080p", "iPhone 6", "Web: Facebook HD 720p 24fps")
  - **Custom Presets**: You define codec, bitrate, resolution, frame rate, audio channels
- You do NOT need to know codec settings manually — apply the right preset.

> **Example Presets:**
> - `1351620000001-000001` → Generic 1080p
> - `1351620000001-100070` → iPhone 5 (1080p)
> - `1351620000001-200045` → Web: 720p
> - `1351620000001-500050` → Audio MP3 → 128k

#### Thumbnail
- Elastic Transcoder can extract **thumbnail images** from the video at defined intervals.
- Stored as JPEG or PNG in the output S3 bucket.
- Used for video preview images in streaming platforms.

---

### Supported Input Formats

| Category | Formats |
|---|---|
| **Video containers** | MP4, MOV, MXF, FLV, AVI, WMV, 3GP, TS, MKV |
| **Video codecs** | H.264, H.265 (HEVC), MPEG-2, VP8, VP9 |
| **Audio** | AAC, MP3, PCM, WAV, FLAC, Vorbis |

---

### Supported Output Formats

| Format | Use Case |
|---|---|
| **MP4 (H.264/AAC)** | Universal; works on all devices |
| **HLS (HTTP Live Streaming)** | iOS, macOS, Smart TVs; adaptive bitrate |
| **MPEG-DASH** | Android, web; adaptive bitrate |
| **WebM (VP8/VP9)** | Web browsers (Chrome, Firefox) |
| **Smooth Streaming** | Microsoft/Silverlight (legacy) |
| **MP3/AAC** | Audio-only output |
| **GIF** | Animated GIF from video clip |

---

### HLS and Adaptive Bitrate Streaming

**HLS (HTTP Live Streaming)** is the most common output for video streaming:

- Video is split into small **segments** (e.g., 10-second .ts files).
- A **manifest file** (.m3u8) lists all segments and available bitrate variants.
- The player automatically switches quality based on network speed.
- Elastic Transcoder creates the HLS segments and manifest file automatically.

```
S3 Input: movie.mp4 (4K, 8 Gbps raw)
    ↓ Elastic Transcoder Job (HLS preset)
S3 Output:
    ├── movie_1080p/  → 10-second .ts segments
    ├── movie_720p/   → 10-second .ts segments
    ├── movie_360p/   → 10-second .ts segments
    └── master.m3u8   → Master playlist referencing all quality levels
```

---

### Elastic Transcoder Architecture

```
Raw Video File
(uploaded to S3)
      ↓
S3 Input Bucket
      ↓
Elastic Transcoder Pipeline
      ↓ (Job submitted via API/Lambda/Console)
Elastic Transcoder Job
  ├── Output 1: 1080p MP4 (System Preset)
  ├── Output 2: 720p HLS Segments
  ├── Output 3: 480p DASH
  └── Thumbnails: every 60 seconds → JPEG
      ↓
S3 Output Bucket
      ↓
CloudFront Distribution
      ↓
End Users (TV, Mobile, Web)
```

---

### Notifications

Elastic Transcoder sends **SNS notifications** on:
- Job **Progressing** (started)
- Job **Complete** (success)
- Job **Warning** (non-fatal issues)
- Job **Error** (failed)

This allows event-driven workflows: SNS → Lambda → update database, trigger CDN invalidation, send user email.

---

### Encryption

| Type | Description |
|---|---|
| **S3 server-side encryption** | Input and output files encrypted at rest in S3 (SSE-S3 or SSE-KMS) |
| **HLS content protection** | AES-128 encryption of HLS segments; key served via HTTPS |
| **DRM** | Not natively supported; use third-party DRM with Elastic Transcoder |

---

### Elastic Transcoder vs AWS Elemental MediaConvert

> **This is the most important comparison for the exam.**

| Feature | Elastic Transcoder | AWS Elemental MediaConvert |
|---|---|---|
| **Status** | Legacy (still available) | **Current, recommended service** |
| **Pricing model** | Per minute of output video | Per minute + feature-based |
| **Broadcast features** | Limited | ✅ Full broadcast-grade (captions, HDR, Dolby) |
| **Ad insertion** | ❌ | ✅ SCTE-35 ad markers |
| **Captions/Subtitles** | Basic | ✅ SRT, TTML, SCC, WebVTT |
| **HDR** | ❌ | ✅ HDR10, Dolby Vision |
| **Reserved queues** | ❌ | ✅ (predictable cost for high volume) |
| **Accelerated transcoding** | ❌ | ✅ (parallel processing) |
| **Use case** | Simple web/mobile transcoding | Broadcast, OTT, professional media |

> **Exam Rule:** For new workloads, AWS recommends **MediaConvert** over Elastic Transcoder. If the exam asks about "simple transcoding from S3 to S3 for web/mobile" → Elastic Transcoder. If it asks about "broadcast-quality", "captions", "HDR", "ad insertion" → MediaConvert.

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Convert MP4 uploaded to S3 into multiple HLS resolutions | Elastic Transcoder |
| Auto-transcode when video uploaded to S3 | S3 Event → Lambda → Elastic Transcoder Job |
| Extract thumbnail images from video | Elastic Transcoder thumbnail feature |
| Get notified when transcoding job completes | Elastic Transcoder SNS notification |
| Broadcast-quality transcoding with captions and HDR | AWS Elemental MediaConvert (not Elastic Transcoder) |
| Serverless video processing pipeline | S3 → Lambda → Elastic Transcoder → S3 → CloudFront |
| Protect HLS video stream from unauthorized access | CloudFront Signed URLs + HLS AES-128 encryption |

---

---

## 2. Amazon Kinesis Video Streams

### What Problem Does It Solve?

Millions of IoT devices, IP cameras, smartphones, drones, and dashcams generate continuous live video. Traditionally, building infrastructure to ingest, store, and process this video at scale requires significant engineering effort — managing servers, handling network interruptions, dealing with device reconnects, storing video durably, and enabling real-time ML inference.

**Kinesis Video Streams solves this by:**
- Providing a managed, scalable service to **ingest live video** from any number of devices
- **Durably storing** video (seconds to years) with time-indexed access
- Enabling **real-time and batch video analytics** using ML services (Rekognition, SageMaker)
- Supporting **WebRTC** for two-way, ultra-low latency video communication

**Real-World Examples:**
> 1. **Smart Home Security:** 500 home cameras stream live video to Kinesis Video Streams. Amazon Rekognition Video analyzes each stream for motion, faces, and package detection in real time. Alerts are sent via SNS when a person is detected at the front door.
>
> 2. **Autonomous Vehicles:** Dashcams on 10,000 fleet vehicles continuously stream video to Kinesis Video Streams. ML models analyze video for near-miss incidents, driver behavior, and road conditions.
>
> 3. **Telehealth Video:** A hospital uses Kinesis Video Streams with WebRTC for real-time two-way video between doctors and patients — with the video durably stored for compliance.

---

### Core Concepts

#### Stream
- The fundamental unit — represents a **single video source** (one camera, one device).
- Named resource; stores video fragments with time-based indexing.
- **Retention period**: Configurable from 0 hours (no storage) to 87,600 hours (10 years).
- Unlimited streams per account.

#### Producer
- A **device or application** that sends video data to Kinesis Video Streams.
- Uses the **Kinesis Video Streams Producer SDK** (C++, Java, Android, iOS).
- Handles: connection, retry logic, buffering, codec negotiation.
- Examples: IP camera, Raspberry Pi, smartphone, drone, IoT device.

#### Consumer
- An **application** that reads and processes video from a stream.
- Types:
  - **Real-time consumer**: Reads live fragments as they arrive
  - **Historical consumer**: Reads stored video from a past timestamp
- Examples: ML inference app, recording application, monitoring dashboard.

#### Fragment
- A self-contained unit of video (typically 2–10 seconds of video + audio).
- Each fragment has a **producer timestamp** and **server timestamp**.
- Fragments are the smallest unit of consumption.
- Stored durably and retrievable by timestamp range.

#### MKV (Matroska) Format
- Kinesis Video Streams uses **MKV** as the streaming container format.
- The Producer SDK handles encoding to MKV automatically.
- Consumers receive MKV fragments.

---

### Kinesis Video Streams Ingestion Flow

```
Physical Devices / Sources
│
├── IP Cameras (RTSP)
├── Smartphones (Android/iOS SDK)
├── Drones / Dashcams
├── Raspberry Pi / IoT devices
└── WebRTC peers
         │
         ↓ (Producer SDK — handles TLS, auth, retry)
Amazon Kinesis Video Streams
         │
    Stream Storage
    (time-indexed, durable, S3-backed internally)
         │
    ┌────┴──────────────────────────┐
    ↓                               ↓
Real-Time Processing          Historical Playback
(Rekognition, Lambda,         (playback at any past
 SageMaker, custom app)        timestamp range)
```

---

### Playback APIs

Kinesis Video Streams provides multiple APIs for reading video:

| API | Description | Use Case |
|---|---|---|
| **GetMedia** | Reads live or historical MKV fragments continuously | Real-time processing apps |
| **GetMediaForFragmentList** | Retrieves specific fragments by fragment number | Replay specific events |
| **ListFragments** | Lists fragments in a time range | Find relevant video segments |
| **GetHLSStreamingSessionURL** | Generates an HLS URL for playback | In-browser video playback |
| **GetDASHStreamingSessionURL** | Generates a DASH URL for playback | Adaptive bitrate playback |

> **Exam Tip:** `GetHLSStreamingSessionURL` is the key API for playing back Kinesis Video Streams in a web browser without writing a custom player.

---

### Kinesis Video Streams + Amazon Rekognition Video

**The most important integration for the exam.**

```
IP Camera Stream
      ↓
Kinesis Video Streams (ingest + store)
      ↓
Amazon Rekognition Video
      ↓ (real-time analysis per fragment)
Kinesis Data Streams (output stream of detections)
      ↓
Lambda Function
      ↓
SNS / DynamoDB / EventBridge
```

**What Rekognition Video can detect:**
- Faces (detection, recognition, comparison)
- Objects and scenes
- Text in video
- People and activity
- Inappropriate content (content moderation)
- PPE (Personal Protective Equipment) detection

**Example:** A retail store with 50 cameras streams to Kinesis Video Streams. Rekognition Video detects when a customer is at an empty checkout lane and alerts staff via SNS in real time.

---

### WebRTC Support

Kinesis Video Streams supports **WebRTC** for ultra-low latency (sub-second) two-way video:

| Feature | Details |
|---|---|
| **Latency** | Sub-second (< 500ms) |
| **Protocol** | WebRTC (ICE, DTLS, SRTP) |
| **Use case** | Two-way video communication (telehealth, remote monitoring, video doorbells) |
| **Signaling** | Kinesis Video Streams provides the WebRTC signaling service |
| **TURN/STUN** | AWS provides TURN and STUN servers for NAT traversal |
| **Peer types** | Browser-to-device, device-to-device, device-to-browser |

**WebRTC vs Standard Streaming:**

| Feature | Standard Streaming | WebRTC |
|---|---|---|
| **Latency** | 2–5 seconds | < 500ms |
| **Direction** | One-way (device → cloud) | Two-way |
| **Storage** | Yes (in stream) | Optional |
| **Use case** | Surveillance, analytics | Live interaction, doorbells |

---

### Kinesis Video Streams vs Kinesis Data Streams

> **Very commonly confused on the exam.**

| Feature | Kinesis Video Streams | Kinesis Data Streams |
|---|---|---|
| **Data type** | Video, audio, RADAR, LIDAR | Any data records (JSON, CSV, logs, events) |
| **Record unit** | Video fragment (MKV, seconds long) | Data record (up to 1 MB) |
| **Ingestion** | Producer SDK on device | Kinesis Producer Library (KPL) or API |
| **Consumers** | Rekognition Video, custom apps | Lambda, KCL, Firehose, Analytics |
| **Storage** | Up to 10 years | 24 hours to 365 days |
| **Shard concept** | ❌ No shards (per-stream) | ✅ Shards (partition key based) |
| **Ordering** | Per stream, time-ordered | Per shard, sequence-ordered |
| **Use case** | Video surveillance, IoT cameras | Event streaming, clickstreams, logs |

---

### Kinesis Video Streams vs Amazon IVS (Interactive Video Service)

| Feature | Kinesis Video Streams | Amazon IVS |
|---|---|---|
| **Use case** | IoT/device video ingestion + analytics | Live streaming to large audiences |
| **Protocol** | SDK-based (Producer SDK), WebRTC | RTMPS ingest |
| **Latency** | Configurable (near-real-time to batch) | < 3 seconds (ultra-low latency) |
| **Audience** | Machines / analytics systems | Human viewers (Twitch-like) |
| **Rekognition integration** | ✅ Native | ❌ |
| **Interactive features** | ❌ | ✅ Timed metadata, chat |
| **Use case example** | Security camera analytics | Live game streaming, virtual events |

---

### Storage and Retention

| Retention Setting | Use Case |
|---|---|
| **0 hours** | No storage; real-time processing only; lowest cost |
| **1–24 hours** | Short-term buffer for real-time processing with recent replay |
| **7 days** | Typical for security camera footage review |
| **30 days** | Insurance/compliance review windows |
| **87,600 hours (10 years)** | Maximum; long-term compliance or forensic use |

> **Cost Consideration:** You pay for: (1) volume of video ingested (per GB), (2) video stored (per GB/month), (3) video consumed/read (per GB). Setting retention to 0 eliminates storage costs when only real-time processing is needed.

---

### Kinesis Video Streams Security

| Feature | Details |
|---|---|
| **Encryption in transit** | TLS 1.2 always enforced |
| **Encryption at rest** | KMS server-side encryption |
| **IAM** | IAM policies control PutMedia, GetMedia per stream |
| **VPC** | Interface endpoint (PrivateLink) for private VPC access |
| **Stream-level IAM** | Fine-grained: different roles for producers vs consumers |

---

### Producer SDK Platforms

| Platform | SDK |
|---|---|
| **C++** | Linux, macOS, Windows, Raspberry Pi, embedded |
| **Java** | JVM-based applications |
| **Android** | Native Android apps |
| **iOS/macOS** | Swift/Objective-C |
| **GStreamer plugin** | IP cameras with RTSP using GStreamer pipeline |

> **Exam Tip:** IP cameras with RTSP feeds connect to Kinesis Video Streams using the **GStreamer plugin** — they don't natively run the SDK but GStreamer acts as the bridge.

---

### Kinesis Video Streams Architecture Patterns

#### Pattern 1: Smart Building Security
```
100 IP Cameras (RTSP)
      ↓ (GStreamer plugin)
Kinesis Video Streams (100 streams, 7-day retention)
      ↓
Rekognition Video (face detection, PPE detection)
      ↓
Kinesis Data Streams (detection events)
      ↓
Lambda → DynamoDB (log events) + SNS (real-time alerts)
```

#### Pattern 2: Video Doorbell (WebRTC)
```
Doorbell Device (camera + mic)
      ↓ WebRTC (sub-second latency)
Kinesis Video Streams (WebRTC signaling + TURN)
      ↓ WebRTC
Mobile App (homeowner sees and speaks to visitor)
      ↓
Optional: Store video clip to stream (motion events)
```

#### Pattern 3: ML Training Data Collection
```
Fleet Vehicles (dashcams, 10,000 trucks)
      ↓ (Producer SDK)
Kinesis Video Streams (30-day retention)
      ↓ (GetMediaForFragmentList — extract interesting clips)
S3 (raw video clips for labeling)
      ↓
SageMaker Ground Truth (human labeling)
      ↓
SageMaker Training Job (train object detection model)
```

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Ingest live video from 1,000 IoT cameras | Kinesis Video Streams |
| Real-time face detection from security cameras | KVS + Rekognition Video + Kinesis Data Streams |
| Play back security camera footage from 3 days ago in browser | KVS GetHLSStreamingSessionURL (historical playback) |
| Two-way video for telehealth with sub-second latency | KVS WebRTC |
| IP camera (RTSP) to Kinesis Video Streams | GStreamer plugin on a local server |
| Store video for exactly 30 days then auto-expire | KVS 30-day retention period |
| Real-time video analytics without storing video | KVS with 0-hour retention + GetMedia consumer |
| Live streaming a gaming event to thousands of viewers | Amazon IVS (not KVS) |

---

---

## Master Comparison: Elastic Transcoder vs Kinesis Video Streams

| Dimension | Elastic Transcoder | Kinesis Video Streams |
|---|---|---|
| **Data source** | Stored files in S3 | Live streams from devices |
| **Processing model** | Batch (job-based) | Real-time / continuous |
| **Latency** | Minutes (offline) | Sub-second to seconds |
| **Output** | Transcoded files in S3 | Live stream / stored fragments |
| **ML integration** | ❌ | ✅ Rekognition Video |
| **WebRTC** | ❌ | ✅ |
| **Primary use** | Video on demand (VOD) | Live video surveillance, IoT, video calls |
| **Consumers** | CloudFront viewers | Real-time analytics apps, humans |
| **Serverless** | ✅ (fully managed) | ✅ (fully managed) |

---

## Full End-to-End Streaming Architecture (Exam Pattern)

### VOD (Video On Demand) Pipeline
```
Content Creator → S3 Upload (raw video, any format)
                        ↓ S3 Event Notification
                   Lambda (submit Elastic Transcoder job)
                        ↓
                  Elastic Transcoder Pipeline
                        ↓ (produces multiple outputs)
                  S3 Output Bucket
                  ├── 1080p HLS segments + manifest
                  ├── 720p HLS segments + manifest
                  ├── 480p MP4
                  └── Thumbnails (JPEG)
                        ↓
                  CloudFront CDN (global distribution)
                        ↓ SNS notification on job complete
                   Lambda → DynamoDB (update video status)
                        ↓
                  End Users (TV / Mobile / Web)
```

### Live Video Analytics Pipeline
```
IoT Cameras / Devices
      ↓ (Producer SDK)
Kinesis Video Streams
      ↓                      ↓
Real-Time Path         Historical Path
(GetMedia)             (GetHLSStreamingSessionURL)
      ↓                      ↓
Rekognition Video     Browser Playback
      ↓
Kinesis Data Streams
      ↓
Lambda / Analytics / Alerts
```

---

## Exam Tips Summary

### Amazon Elastic Transcoder
- ✅ Converts stored S3 video files → multiple output formats in S3
- ✅ Pipeline = job queue; Job = single transcode request; Preset = output format template
- ✅ One job → multiple outputs (1080p + 720p + 480p + thumbnails simultaneously)
- ✅ HLS output = segments (.ts files) + manifest (.m3u8) for adaptive streaming
- ✅ SNS notifications for job completion → trigger Lambda for event-driven workflows
- ✅ System presets (pre-built) vs Custom presets (user-defined)
- ✅ **Legacy service** — MediaConvert is the modern replacement for broadcast/professional
- ✅ Use Elastic Transcoder for: simple web/mobile transcoding, S3-to-S3 pipelines
- ✅ Use MediaConvert for: captions, HDR, ad markers, broadcast-quality, Dolby audio

### Amazon Kinesis Video Streams
- ✅ Ingest live video from IoT devices, cameras, mobile apps
- ✅ Fragment = self-contained MKV unit of video (2–10 seconds)
- ✅ Producer SDK handles connection, retry, buffering automatically
- ✅ Retention 0 hours (real-time only) to 10 years (compliance)
- ✅ GetHLSStreamingSessionURL = play back video in browser (key API!)
- ✅ WebRTC = two-way, sub-second latency video communication
- ✅ KVS + Rekognition Video = real-time video analytics (most exam pattern)
- ✅ IP cameras (RTSP) → KVS via GStreamer plugin
- ✅ KVS ≠ Kinesis Data Streams: KVS = video/audio; KDS = general data records
- ✅ KVS ≠ Amazon IVS: KVS = IoT/analytics; IVS = broadcast to human viewers
- ✅ VPC interface endpoint for private stream access (no internet)
- ✅ KMS encryption at rest; TLS always in transit
