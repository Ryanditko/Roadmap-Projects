# Backend for Streaming Platform (like Netflix)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Design the backend for a video streaming service: it ingests a raw upload, transcodes it into multiple resolutions and bitrates, publishes those renditions to a CDN, and serves clients a manifest they can stream adaptively. The hard part is not any single feature — it is orchestrating a long-running asynchronous pipeline, keeping playback smooth over unreliable networks, and tracking millions of concurrent sessions without hammering your database. You will build the control plane (metadata, catalog, session state) and the data plane contract (manifests and segment URLs), while treating storage and CDN as pluggable infrastructure. The focus is architecture and trade-offs, not shipping your own codec.

## Prerequisites

- Solid REST API design and asynchronous job/queue experience (a message-broker or worker-queue project is a good warm-up)
- Familiarity with HTTP caching, CDNs, and range requests
- Comfort with object storage (S3-compatible) and background workers
- Conceptual understanding of video: containers, codecs, bitrate, keyframes
- A framework of your choice (Node, Go, Java, Python) plus any transcoder wrapping FFmpeg

## Learning Objectives

By the end, you should be able to:

- Model a transcoding pipeline as idempotent, resumable, observable stages
- Explain adaptive bitrate streaming and generate HLS/DASH manifests
- Design a catalog/metadata store that scales reads independently of writes
- Track playback sessions (resume position, heartbeat) cheaply at scale
- Reason about CDN offload, cache keys, and signed/expiring URLs
- Choose consistency and availability trade-offs for each subsystem

## Functional Requirements

1. The system must accept a video upload and return an asset ID while transcoding proceeds asynchronously.
2. Transcoding must produce multiple renditions (e.g. 240p–1080p) and segment them for HLS and/or DASH.
3. Each pipeline stage must be idempotent and resumable so a retried or crashed job never duplicates or corrupts output.
4. The system must serve a manifest that lists available renditions; clients pick a bitrate adaptively.
5. Segment and manifest delivery must be CDN-frontable, using signed, expiring URLs for access control.
6. The system must persist per-user playback sessions (resume position, watch history) via lightweight heartbeats.
7. The catalog API must remain available for browsing even if the transcoding pipeline is degraded.
8. **Non-functional:** target ≥99.9% availability for playback and catalog reads; sustain thousands of concurrent heartbeats; keep manifest latency low via caching; define behavior under partial CDN failure and worker backlog.

## Suggested Milestones

1. **Milestone 1 — Upload & metadata:** Accept an upload to object storage, create the asset record, enqueue a transcode job.
2. **Milestone 2 — Transcoding pipeline:** Run staged workers (probe → transcode renditions → segment → generate manifest), each idempotent and status-tracked.
3. **Milestone 3 — Playback:** Serve manifests and CDN-signed segment URLs; implement session heartbeats and resume.
4. **Milestone 4 — Scale & resilience:** Add caching, retries with backoff, dead-letter handling, and metrics/dashboards for pipeline health.

## Data & Interface Sketch

```text
Components
  [Client] --upload--> [Ingest API] --> [Object Storage (raw)]
                                   \--> [Job Queue] --> [Transcode Workers]
                                                             |
                                        [Object Storage (renditions + segments)]
                                                             |
  [Client] <--manifest/segments-- [CDN] <---- origin ------ [Delivery API]
  [Client] --heartbeat--> [Session API] --> [KV / cache] --async--> [History DB]

Asset
  id, title, status(uploaded|transcoding|ready|failed),
  durationSec, renditions[{height, bitrate, codec, manifestUrl}]

GET  /catalog/{id}          -> asset metadata
GET  /play/{id}/manifest    -> HLS(.m3u8) or DASH(.mpd), signed segment URLs
POST /sessions/{id}/heartbeat  body:{ positionSec } -> 204
GET  /sessions/{id}         -> { resumePositionSec, updatedAt }
```

## Stretch Goals

- Add subtitle/caption tracks (WebVTT) referenced in the manifest.
- Implement a simple recommendation feed from watch history.
- Add per-title encoding (choose bitrate ladder based on content complexity).
- Support live streaming with a rolling segment window.

## Definition of Done

- [ ] An upload transitions through pipeline states to `ready` with multiple renditions produced.
- [ ] A retried or duplicated transcode job produces no duplicate or partial output.
- [ ] A player can fetch a valid HLS or DASH manifest and stream segments via signed CDN URLs.
- [ ] Resume position survives disconnect and is restored on next play.
- [ ] Catalog reads succeed while the transcoding pipeline is intentionally paused.
- [ ] Pipeline metrics (queue depth, stage latency, failure rate) are observable.

## Common Pitfalls

- Making transcode stages non-idempotent, so a retry re-appends segments or double-charges storage.
- Blocking the upload request on transcoding instead of returning immediately and working asynchronously.
- Writing every heartbeat straight to the primary database and melting it under load — buffer in a cache.
- Serving segments from origin instead of the CDN, defeating the whole point of edge caching.
- Forgetting URL/segment expiry, leaking permanent access to protected content.

## Resources

- [Apple: HTTP Live Streaming (HLS)](https://developer.apple.com/documentation/http-live-streaming) — the canonical HLS reference.
- [MPEG-DASH overview (Bitmovin)](https://bitmovin.com/mpeg-dash-explained/) — DASH manifests and adaptive streaming.
- [Netflix TechBlog: Per-Title Encode Optimization](https://netflixtechblog.com/per-title-encode-optimization-7e99442b62a2) — real-world bitrate ladder trade-offs.
- [FFmpeg documentation](https://ffmpeg.org/documentation.html) — transcoding and segmentation.
- [MDN: HTTP range requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Range_requests) — partial content delivery for media.
