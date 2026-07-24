# Design a CDN

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Design a Content Delivery Network — the fleet of edge servers that sits between users and origin infrastructure to serve static and streamable content (images, JS/CSS bundles, video segments) from a location physically close to each visitor. The interesting design tension is not "put a cache near the user" but everything that follows: how you route a request to the right edge, how you keep cached copies fresh without hammering the origin, and how you purge a bad asset from hundreds of points of presence in seconds. This is a design exercise — the deliverable is a written design document with diagrams, not a running CDN.

## Prerequisites

- Comfort with HTTP caching semantics (see [Design a Rate Limiter](../../beginner/06-rate-limiter/) or an intermediate caching project as a warm-up)
- Understanding of DNS resolution and how clients pick a server
- Familiarity with cache headers: `Cache-Control`, `ETag`, `Last-Modified`, TTL
- Basic capacity/back-of-the-envelope estimation (requests/sec, bandwidth, storage)

## Learning Objectives

By the end, you should be able to:

- Estimate edge capacity from traffic, object size, and cache hit ratio, and justify the numbers
- Compare request-routing strategies (DNS-based, Anycast, GeoDNS) and pick one with reasons
- Design a cache freshness and invalidation strategy, including near-instant purge
- Partition the object namespace across edge nodes (consistent hashing) and explain rebalancing
- Articulate at least two design trade-offs with the alternative you rejected and why

## Requirements & Constraints

**Functional**

- Serve cacheable content from the edge closest to the user, falling back to origin on a miss.
- Support cache invalidation: purge a specific object or a path prefix globally within seconds.
- Honor origin cache directives (TTL, `no-store`, `ETag` revalidation).
- Expose a customer-facing API to configure origins, cache rules, and trigger purges.

**Non-functional**

- Target cache hit ratio ≥ 90% for static assets; edge response p99 < 50 ms on a hit.
- Availability 99.9%+; an edge failure must not take down content delivery for its region.
- Protect the origin from thundering-herd on cold cache or mass invalidation.
- Cost-aware: minimize origin egress and inter-PoP bandwidth.

## Suggested Approach

1. **Estimate first.** Pick a scenario (e.g. 50 TB/day egress, 5 KB average object, 90% hit ratio). Derive peak requests/sec, per-PoP bandwidth, and hot-set storage. Let these numbers drive PoP count and cache sizing.
2. **Choose request routing.** Compare GeoDNS vs. Anycast for steering users to a PoP. Note DNS TTL caching effects and failover latency.
3. **Design the cache tier.** Decide eviction policy (LRU vs. LFU vs. TinyLFU) and a two-tier layout (edge → regional shield → origin) to absorb misses.
4. **Design invalidation.** Choose between short TTL, versioned URLs, and active purge. Sketch how a purge fans out to all PoPs.
5. **Partition the namespace.** Use consistent hashing so objects map to nodes with minimal reshuffling when a node joins or leaves.

## Architecture Sketch

```text
                 ┌─────────── Control plane ───────────┐
   Customer ───► │  Config API  ·  Purge API  ·  Metrics │
                 └───────────────────┬──────────────────┘
                                     │ (push config / purge)
   User ─DNS/Anycast─► PoP (edge) ──miss──► Regional shield ──miss──► Origin
        (nearest)      │  LRU/TinyLFU cache        │  larger cache
                       └──hit──► response          └──hit──► response

Object (cache entry)
  key:        hash(host + path + variant)
  bytes:      blob
  ttl:        seconds       (from origin Cache-Control)
  etag:       string        (for revalidation)
  storedAt:   epoch ms

Customer API
  PUT  /v1/distributions/{id}      body: { origin, cacheRules[] }
  POST /v1/distributions/{id}/purge body: { paths: ["/img/*"] } -> 202 { purgeId }
  GET  /v1/distributions/{id}/stats -> { hitRatio, egressBytes, p99Ms }

Routing: user -> GeoDNS resolves to nearest PoP VIP
Partitioning: consistent hash ring maps object key -> edge node
```

## Deep-Dive Topics

- **Cache hierarchy & shields:** Why a mid-tier shield cuts origin load, and how it changes the effective hit ratio.
- **Invalidation semantics:** Purge-by-path fan-out vs. cache-busting via versioned URLs — latency and correctness trade-offs.
- **Consistent hashing:** Virtual nodes for even load; behavior when a PoP is added or fails.
- **Cache stampede prevention:** Request coalescing / single-flight and stale-while-revalidate.
- **Security at the edge:** TLS termination, signed URLs, and basic DDoS absorption.

## Deliverables

- A design document (~2–4 pages) covering routing, caching, partitioning, and invalidation.
- Capacity estimation: peak RPS, per-PoP bandwidth, and hot-set storage, with assumptions stated.
- The architecture diagram, data model, and customer API contract.
- At least two justified trade-offs (e.g. GeoDNS vs. Anycast; short TTL vs. active purge), naming the rejected option and why.

## Common Pitfalls

- Reporting a hit ratio without estimating the hot-set size that makes it achievable.
- Choosing DNS-based routing without accounting for resolver TTL caching, which slows failover.
- Treating invalidation as an afterthought — global purge latency is a core CDN metric, not a footnote.
- Ignoring the thundering-herd on cold cache; without coalescing, one popular miss can overwhelm the origin.
- Picking modulo hashing for partitioning, which reshuffles almost everything when node count changes.

## Resources

- [Cloudflare: What is a CDN?](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/) — clear primer on edge delivery and PoPs.
- [MDN: HTTP caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching) — `Cache-Control`, `ETag`, and revalidation semantics.
- [System Design Primer: CDN](https://github.com/donnemartin/system-design-primer#content-delivery-network) — trade-offs of push vs. pull CDNs.
- [Consistent Hashing (original paper)](https://www.cs.princeton.edu/courses/archive/fall09/cos518/papers/chash.pdf) — the basis for object-to-node partitioning.
- [web.dev: Network reliability](https://web.dev/articles/http-cache) — practical caching strategy guidance.
