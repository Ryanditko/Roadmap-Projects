# API with Caching Layer (Redis)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

An API that reads the same rarely-changing data from the database on every request is doing a lot of expensive work for no reason. This project puts a Redis cache in front of that work: the first request pays the full cost and stores the result, and the next thousand requests are answered from memory in a fraction of the time. The genuinely interesting part is not the read path — it is everything that keeps the cache honest. When does a cached value expire? What happens when the underlying data changes? What do you do when a thousand requests all miss at the same instant? Getting caching right is mostly the discipline of never serving data you can no longer trust.

## Prerequisites

- An existing API with at least one read-heavy endpoint backed by a datastore ([Blog Platform with CRUD and Comments](../02-blog-platform-crud/) is a good candidate)
- A running Redis instance and a client library for your language
- Understanding of TTL, serialization (JSON or binary), and basic Big-O of your queries
- Comfort measuring latency so you can prove the cache actually helped

## Learning Objectives

By the end, you should be able to:

- Implement the cache-aside (lazy-loading) pattern correctly, including the miss path
- Choose sensible TTLs and reason about staleness versus freshness trade-offs
- Invalidate or update cache entries when the source data changes
- Design a consistent, collision-free key naming scheme
- Detect and mitigate cache stampede (thundering herd) on a hot key
- Measure hit/miss ratio and memory use to know whether the cache is worth it

## Functional Requirements

1. A cacheable read must check Redis first and only hit the database on a miss.
2. On a miss, the fetched value must be written back to the cache with a TTL before being returned.
3. A write, update, or delete of a resource must invalidate (or refresh) its cached representation so stale data is never served after a known change.
4. Cache keys must follow a documented, collision-free naming scheme that encodes the resource and any parameters.
5. The system must expose cache statistics — at minimum hit count, miss count, and hit ratio.
6. A Redis outage must degrade gracefully to the database, never returning a 500 purely because the cache is down.
7. Concurrent misses on the same key must not all stampede the database.

## Suggested Milestones

1. **Milestone 1 — Read-through cache:** Implement cache-aside on one endpoint with TTL and a clean key scheme; measure the latency win.
2. **Milestone 2 — Invalidation:** Wire writes to invalidate or update affected keys and confirm no stale reads follow a change.
3. **Milestone 3 — Resilience & visibility:** Add stampede protection, graceful fallback on Redis failure, and a stats endpoint.

## Data & Interface Sketch

```text
Key scheme:  <resource>:<version>:<id>[:<paramHash>]
             e.g. "post:v1:42"   "posts:v1:list:tag=redis&page=2"

Cache-aside read:
  value = redis.GET(key)
  if value == nil:                       # miss
      value = db.load(...)
      redis.SET(key, serialize(value), EX=ttl)
  return deserialize(value)

Invalidation on write:
  db.update(id, ...)
  redis.DEL("post:v1:42")                # or SET with fresh value
  redis.DEL(matching list keys)          # bump version to drop many at once

Stampede guard: short lock (SET NX) OR stale-while-revalidate OR jittered TTL

GET  /posts/{id}      -> 200 (X-Cache: HIT|MISS)
GET  /admin/cache     -> 200 { hits, misses, hitRatio, keys, memoryMb }
```

## Stretch Goals

- Implement stale-while-revalidate: serve the expired value once while refreshing in the background.
- Add cache warming on startup for the hottest keys.
- Use versioned key prefixes so you can invalidate an entire class of entries by bumping a version counter.
- Add per-endpoint TTL configuration and expose Redis memory-eviction policy (`maxmemory-policy`) tuning.

## Definition of Done

- [ ] A repeated read is measurably faster on the second call and reports `X-Cache: HIT`.
- [ ] Updating a resource is immediately reflected on the next read — no stale value survives a known write.
- [ ] Killing Redis mid-run still returns correct responses from the database.
- [ ] The stats endpoint reports a hit ratio that rises under repeated reads.
- [ ] A burst of concurrent misses on one cold key triggers at most one database load.

## Common Pitfalls

- Caching without an invalidation plan, so users see data that changed minutes ago.
- Forgetting to set a TTL, letting the cache grow until Redis evicts unpredictably.
- Caching error responses or empty results and then serving them long after the data exists (negative-cache carefully).
- Building keys from unsanitized user input, causing collisions or unbounded key growth.
- Treating a Redis timeout as fatal instead of falling back to the source of truth.

## Resources

- [Redis: Caching patterns and best practices](https://redis.io/docs/latest/develop/use/patterns/) — cache-aside, write-through, and eviction guidance from the source.
- [AWS: Caching strategies (Lazy Loading vs Write-Through)](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/Strategies.html) — a clear comparison with trade-offs.
- [Redis: Key eviction policies](https://redis.io/docs/latest/develop/reference/eviction/) — how `maxmemory-policy` and LRU/LFU actually behave.
- [Facebook: Scaling Memcache (cache stampede, thundering herd)](https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf) — the classic paper on caching at scale.
