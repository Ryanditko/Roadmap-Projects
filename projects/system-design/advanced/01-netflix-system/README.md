# Design a Netflix-like Video Streaming Platform

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Advanced · **Estimated time:** 3–7 days

## Overview

Design a global on-demand video streaming service in the style of Netflix: users browse a personalized catalog, press play, and receive a smooth, adaptive stream that survives network hiccups. The interesting engineering is not the player — it is everything behind it: ingesting a master file, transcoding it into dozens of bitrate/resolution renditions, pushing those bytes to edge caches close to viewers, and serving a home page whose rows are ranked per user in tens of milliseconds. This is a design exercise: your deliverable is a written design document with diagrams and trade-off analysis, not running code.

## Prerequisites

- Solid grasp of HTTP, TCP, and TLS, plus how CDNs and edge caching work
- Familiarity with the read-heavy scaling toolkit (replicas, caches, sharding)
- Basic video concepts: codecs, containers, adaptive bitrate (HLS/DASH)
- Exposure to a lower-level design first (browse the `../` intermediate catalog projects) helps

## Learning Objectives

By the end, you should be able to:

- Estimate storage, egress bandwidth, and transcoding compute for a catalog at planet scale
- Design an offline transcoding pipeline that produces an ABR ladder per title
- Reason about CDN placement, cache hit ratios, and origin shielding
- Separate the personalization/ranking path from the streaming data plane
- Choose consistency models appropriate to catalog metadata versus viewing history

## Requirements & Constraints

1. Serve a personalized home page (rows of titles) with p99 latency under ~200 ms.
2. Support adaptive streaming that adjusts rendition to available bandwidth without rebuffering.
3. Target 99.99% availability for playback; degraded personalization is acceptable, a dead player is not.
4. Assume ~250M subscribers, ~10% peak concurrency, and a multi-terabyte catalog.
5. Enforce DRM and geo/licensing restrictions per title and region.
6. Optimize for egress cost — bandwidth dominates the bill at this scale.
7. Capture viewing telemetry (resume points, QoE) without blocking playback.

## Suggested Approach

Start with capacity math: derive peak concurrent streams, average bitrate, and therefore aggregate egress (Tbps). Size catalog storage after multiplying each title by its rendition ladder. Then split the system into three planes: a **control plane** (catalog metadata, entitlements), a **data plane** (segment delivery via CDN), and an **intelligence plane** (recommendations, ranking). Design the ingest/transcode pipeline as an asynchronous, idempotent job graph. Treat the CDN as your primary scaling lever and the origin as a shielded fallback. Precompute personalized rows offline and serve them from a fast store, refreshing on a cadence rather than per request.

## Architecture Sketch

```text
Client player ──> API Gateway ──> Home/Ranking svc ──> Precomputed rows (KV cache)
                       │
                       ├──> Catalog svc ──> Metadata DB (replicated)
                       └──> Entitlement/DRM svc ──> license server

Play request: player ──> Steering svc ──> nearest CDN PoP ──> segment (.ts/.m4s)
                                              └─ miss ─> Origin shield ─> Object store (S3-like)

Ingest: master ──> Transcode pipeline (job graph) ──> ABR ladder ──> Object store ──> CDN fill

Key APIs:
GET  /home?profile=P           -> { rows: [ {title, ranked_items[]} ] }
GET  /titles/{id}/manifest     -> HLS/DASH manifest (per-region, per-device)
POST /playback/heartbeat       -> { titleId, positionMs, bitrate, droppedFrames }

Data model (sketch):
Title{ id, metadata, availabilityByRegion[], renditions[] }
Rendition{ resolution, bitrate, codec, segmentUri }
ViewHistory{ profileId, titleId, positionMs, updatedAt }  # last-write-wins
```

## Deep-Dive Topics

- **ABR ladder design:** how many renditions, which resolutions/bitrates, per-title encoding.
- **CDN economics:** cache hierarchy, hit-ratio targets, origin shield, ISP-embedded caches.
- **Personalization at scale:** offline batch ranking vs online re-ranking; cold-start users.
- **Consistency split:** strong-ish for entitlements, eventual for view history and rankings.
- **DRM & licensing:** key delivery, regional blackout enforcement, token-signed segment URLs.

## Deliverables

- [ ] A design document (~4–8 pages) with the architecture diagram above, refined.
- [ ] Back-of-envelope capacity estimates for storage, peak egress, and transcode compute.
- [ ] Explicit consistency choices per data domain, with justification.
- [ ] A failure/DR analysis: PoP outage, region loss, origin failure, transcode backlog.
- [ ] Identified bottlenecks and hotspot mitigations (e.g. viral new release thundering herd).

## Common Pitfalls

- Routing every playback byte through your origin instead of the CDN — the bill and the origin both explode.
- Recomputing personalized rows synchronously per request, blowing the latency budget.
- Treating view-history writes as strongly consistent; they should be async and idempotent.
- Ignoring the transcode backlog: a large catalog import can starve encoding for days.
- Forgetting regional licensing, so a title plays where it is not licensed.

## Resources

- [Netflix Tech Blog](https://netflixtechblog.com/) — first-hand accounts of the real architecture.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — CDN, caching, and scaling fundamentals.
- [HTTP Live Streaming (RFC 8216)](https://datatracker.ietf.org/doc/html/rfc8216) — the HLS adaptive streaming spec.
- [Open Connect (Netflix CDN)](https://openconnect.netflix.com/en/) — how Netflix embeds caches inside ISPs.
- [MPEG-DASH overview (MDN)](https://developer.mozilla.org/en-US/docs/Web/Media/DASH_Adaptive_Streaming_for_HTML_5_Video) — adaptive streaming on the web.
