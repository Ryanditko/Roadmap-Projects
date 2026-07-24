# Design a YouTube-like Video Sharing Platform

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Advanced · **Estimated time:** 3–7 days

## Overview

Design a user-generated video platform in the style of YouTube: anyone can upload, the system transcodes and publishes it globally, and viewers watch, search, and get recommended what to watch next. Unlike a curated catalog, the ingest side is a firehose — hundreds of hours uploaded every minute — and the read side is planetary. The design must balance an unbounded upload pipeline, a search index over billions of items, a recommendation engine, and CDN delivery. This is a design exercise: your deliverable is a written design document with diagrams and trade-off analysis, not running code.

## Prerequisites

- Understanding of CDNs, object storage, and adaptive streaming (HLS/DASH)
- Familiarity with asynchronous pipelines and message queues
- Exposure to search indexing (inverted index, ranking) concepts
- Basic understanding of recommendation systems

## Learning Objectives

By the end, you should be able to:

- Design a resumable, chunked upload pipeline feeding an async transcode fan-out
- Estimate ingest storage growth and read egress at global scale
- Design a search index that stays fresh as new videos land continuously
- Separate view-count aggregation from the hot read path
- Reason about recommendation serving latency and the freshness/cost trade-off

## Requirements & Constraints

1. Accept uploads of arbitrary size with resumable, chunked transfer.
2. Transcode each upload into an ABR ladder asynchronously; publish when ready.
3. Serve playback via CDN with p99 start latency under ~1 s.
4. Provide search over billions of videos with sub-second query latency.
5. Aggregate view counts and engagement without blocking playback (eventual consistency OK).
6. Assume ~500 hours uploaded/minute and billions of daily views.
7. Support content moderation and copyright matching in the ingest path.

## Suggested Approach

Separate the write-heavy ingest plane from the read-heavy serving plane. Uploads land as chunks in object storage; a completion event fans out transcode jobs (idempotent, retriable) that produce the ABR ladder and thumbnails. Metadata publishes to a search index asynchronously. On the read side, treat the CDN as the delivery workhorse and precompute recommendation candidates offline, re-ranking a small set online. View counts are the classic hotspot: buffer increments in a stream and aggregate them, accepting eventual consistency, so a viral video does not hammer a single row. Run moderation/copyright as async checks that can retract a video post-publish.

## Architecture Sketch

```text
Uploader ──chunks──> Upload svc ──> Object store (raw) ──event──> Transcode fan-out (queue)
                                                                       │
                                                          ABR ladder + thumbnails -> Object store -> CDN
                                                                       │
                                                          Metadata -> Search index (async) + Catalog DB

Watch: viewer ──> API ──> Catalog ──> manifest ──> CDN segments
View counts: player ──beacon──> stream ──aggregate──> counts store (eventual)
Recs: offline candidate gen -> feature store -> online re-rank -> feed

Key APIs:
POST /uploads (resumable)          -> { uploadId, uploadUrls[] }
GET  /videos/{id}                  -> { metadata, manifestUrl, status }
GET  /search?q=...                 -> { results[], nextPage }
POST /videos/{id}/view             -> 202 (async count)

Data model (sketch):
Video{ id, ownerId, status: PROCESSING|LIVE|BLOCKED, renditions[], stats }
Stats{ views, likes, watchTimeMs }   # eventually consistent aggregates
```

## Deep-Dive Topics

- **Resumable uploads:** chunking, retry, checksum verification, multipart to object storage.
- **Transcode fan-out:** priority queues, idempotency, handling the long tail of formats.
- **Search freshness:** near-real-time indexing vs batch; ranking signals.
- **View-count hotspots:** stream aggregation, sharded counters, approximate counting.
- **Moderation & copyright:** content fingerprinting, async takedown, post-publish retraction.

## Deliverables

- [ ] A design document (~4–8 pages) with the ingest and serving planes separated, refined.
- [ ] Capacity estimates for ingest storage growth, transcode compute, and read egress.
- [ ] The view-count aggregation strategy with its consistency trade-off stated.
- [ ] A failure/DR analysis: transcode backlog, search-index lag, CDN PoP loss.
- [ ] A hotspot mitigation plan for a video going viral within minutes.

## Common Pitfalls

- Blocking upload completion on transcoding instead of publishing "processing" and finishing async.
- Incrementing a single view-count row per view — a viral video creates a write hotspot.
- Making search synchronously consistent with uploads, coupling two very different workloads.
- Serving playback from origin instead of CDN, exploding egress cost.
- Skipping idempotency in transcode jobs, so retries produce duplicate renditions.

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — pipelines, CDNs, and search at scale.
- [Google Research: The YouTube recommendation system](https://research.google/pubs/pub45530/) — deep learning for recommendations.
- [Netflix Tech Blog](https://netflixtechblog.com/) — transcoding and adaptive-streaming practices that transfer directly.
- [HTTP Live Streaming (RFC 8216)](https://datatracker.ietf.org/doc/html/rfc8216) — the HLS spec for delivery.
- [Designing Data-Intensive Applications](https://dataintensive.net/) — stream processing and aggregation.
