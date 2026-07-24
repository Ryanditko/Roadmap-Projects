# Design a Caching System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Design a caching layer that sits between an application and a slow backing store, serving hot data from memory to cut latency and load. The heart of the problem is not storing data — it's deciding what to evict when memory fills and how to keep cached values from going stale. You'll reason about eviction policies, read/write patterns, and the classic hazards: stale reads, cache stampedes, and the thundering herd on a cold cache. Deliver a design document covering the cache placement, eviction policy, and invalidation strategy.

## Prerequisites

- Understanding that memory is fast and small, disk/DB is slow and large
- Awareness of what a cache hit and a cache miss are
- Familiarity with the idea of TTL (time to live)
- Comfort reasoning about consistency between a cache and its source of truth

## Learning Objectives

By the end, you should be able to:

- Choose an eviction policy (LRU, LFU, TTL) and justify it
- Compare read/write caching patterns (cache-aside, write-through)
- Reason about staleness and invalidation
- Estimate the hit rate needed to meet a latency goal
- State a trade-off between freshness and performance

## Requirements & Constraints

1. Serve reads from cache on a hit; fall through to the store on a miss.
2. Evict entries when the cache reaches its memory limit.
3. Keep values reasonably fresh — bounded staleness via TTL or invalidation.
4. Handle a cache miss without collapsing the backing store under a stampede.
5. Expose hit/miss metrics so effectiveness is observable.
6. Estimate scale: 100K reads/s, working set 10 GB, target hit rate ≥ 90%.

## Suggested Approach

1. Decide the caching pattern: cache-aside (app manages the cache) is the common default.
2. Do the math: at 100K reads/s and a 90% hit rate, only ~10K reads/s reach the store — that's the load you must survive.
3. Choose an eviction policy and reason about which fits the access pattern.
4. Design invalidation: TTL expiry, explicit invalidation on write, or write-through.
5. Address the stampede: how many requests hit the store when a hot key expires, and how to dampen it.

## Architecture Sketch

```text
              read
Client ──> [ App ] ──> [ Cache ] --hit--> value
                          │ miss
                          ▼
                     [ Store ] -> populate cache -> value

Patterns
  Cache-aside     app reads cache, on miss loads store then writes cache
  Write-through   writes go to cache and store together (fresher, slower writes)
  Read-through    cache itself loads from store on miss

Eviction
  LRU  evict least recently used   LFU  evict least frequently used
  TTL  expire after fixed time

Data model
  cache entry: key | value | expires_at | last_access
```

## Deep-Dive Topics

- **Eviction policies:** when LRU wins vs. LFU, and why TTL alone can serve stale data.
- **Cache stampede:** locking, request coalescing, or probabilistic early expiry to stop many misses hitting the store at once.
- **Consistency:** cache-aside's stale-write race and how write-through or invalidation reduces it.

## Deliverables

- An architecture diagram showing cache placement and the miss path to the store.
- The read/write pattern and eviction policy you chose, with a short rationale.
- A data model for a cache entry.
- One trade-off written up: e.g., write-through (always fresh, slower writes) vs. cache-aside with TTL (fast writes, bounded staleness window).

## Common Pitfalls

- Designing storage but never stating the eviction policy, so behavior at capacity is undefined.
- Ignoring the stampede: a popular key expiring lets thousands of misses hit the store at once.
- Caching without an invalidation plan, so writes leave stale values readable indefinitely.
- Reporting no hit/miss metrics, so you can't tell if the cache helps at all.

## Resources

- [System Design Primer: Cache](https://github.com/donnemartin/system-design-primer#cache) — caching patterns and placement.
- [Redis: Key eviction policies](https://redis.io/docs/latest/develop/reference/eviction/) — how a real cache evicts.
- [AWS: Caching strategies](https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/Strategies.html) — cache-aside vs. write-through.
- [Wikipedia: Cache stampede](https://en.wikipedia.org/wiki/Cache_stampede) — the thundering-herd problem and mitigations.
