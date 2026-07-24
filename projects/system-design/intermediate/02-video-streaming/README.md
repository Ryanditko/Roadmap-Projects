# Design a Video Streaming System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Design the backend for a video-on-demand platform like Netflix or YouTube: creators upload a file, the system transcodes it into multiple qualities, and millions of viewers stream it smoothly on flaky networks. The interesting problems are the offline transcoding pipeline, adaptive bitrate delivery over HTTP, and serving petabytes of bytes cheaply through a CDN. This is a design exercise: you produce a design document, capacity numbers, and diagrams — not working code.

## Prerequisites

- Understanding of HTTP, object storage, and how CDNs cache content
- Basic awareness of video codecs, containers, and bitrate
- Familiarity with message queues and background job processing
- Comfort estimating storage and bandwidth at scale

## Learning Objectives

By the end, you should be able to:

- Design an asynchronous transcoding pipeline driven by a job queue
- Explain adaptive bitrate streaming (HLS/DASH) and manifest/segment layout
- Estimate storage per title across renditions and egress bandwidth at peak
- Design a CDN + origin caching hierarchy and reason about hit ratios
- Justify trade-offs between pre-transcoding all renditions vs. on-demand

## Requirements & Constraints

- Assume 100k new videos/day (avg 10 min), 50M daily viewers, 5M concurrent at peak.
- Playback must start in under ~2 s and adapt smoothly to bandwidth drops.
- Each title is stored in ~5 renditions (240p–1080p) plus thumbnails and subtitles.
- Estimate raw + transcoded storage per title and total peak egress in Tbps.
- Uploads and transcoding are asynchronous; viewing is read-heavy and latency-sensitive.

## Suggested Approach

1. Separate the write path (upload → transcode) from the read path (manifest → segments).
2. Size storage: raw size × renditions × new videos/day; project one year.
3. Design the transcoding pipeline: chunk the source, fan out jobs, reassemble.
4. Design HLS/DASH delivery: manifest points at segments cached at the edge.
5. Plan CDN tiers (edge → regional → origin) and how popularity drives caching.

## Architecture Sketch

```text
Upload -> API -> Raw Object Store (S3-like)
                    |-> transcode jobs -> Queue -> Worker fleet -> Rendition Store + Manifests
                                                        |-> metadata DB (partition by videoId)

Viewer -> CDN edge -> regional cache -> origin (Rendition Store)
   GET /videos/{id}/master.m3u8   -> 200 manifest (rendition list)
   GET /videos/{id}/1080p/seg_{n}.ts -> 200 segment (cached at edge)

POST /videos                 { title, ownerId } -> 201 { videoId, uploadUrl }
GET  /videos/{id}/play                          -> 200 { manifestUrl, subtitles[] }

Video { videoId, ownerId, status, durationSec, renditions[], ts } // partition by videoId
Segment layout: master.m3u8 -> {rendition}.m3u8 -> seg_0.ts ... seg_n.ts
```

## Deep-Dive Topics

- **Transcoding pipeline:** chunk-level parallelism, retries, priority for popular uploads.
- **Adaptive bitrate:** client-driven rendition switching; segment duration trade-offs.
- **Trade-off 1 — pre-transcode all vs. on-demand:** pre-transcoding wastes storage on unwatched titles but guarantees instant playback; on-demand saves storage but adds first-view latency. Justify a hybrid (pre-transcode top renditions, lazy-generate the rest).
- **Trade-off 2 — segment length:** short segments adapt faster to bandwidth changes but bloat manifests and request counts.

## Deliverables

- [ ] A design document (~3–5 pages) covering write and read paths.
- [ ] Capacity estimates: storage per title, total storage/year, peak egress bandwidth.
- [ ] A CDN caching strategy with expected hit ratio and eviction policy.
- [ ] A metadata partitioning plan.
- [ ] At least two trade-offs, each with the option chosen and why.

## Common Pitfalls

- Transcoding the whole file in one job, so a 4-hour movie blocks a worker for hours.
- Ignoring egress cost — bandwidth, not storage, usually dominates the bill.
- Caching manifests as aggressively as segments, so rendition changes never propagate.
- Forgetting thumbnails, subtitles, and audio tracks in the storage estimate.

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — CDN and capacity fundamentals.
- [HLS overview (Apple, RFC 8216)](https://datatracker.ietf.org/doc/html/rfc8216) — the HTTP Live Streaming protocol.
- [Netflix Tech Blog: encoding](https://netflixtechblog.com/) — real-world transcoding and delivery at scale.
- [web.dev: Fast playback with audio and video preload](https://web.dev/articles/fast-playback-with-preload) — startup latency techniques.
