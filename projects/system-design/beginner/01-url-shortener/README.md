# Design a URL Shortener

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Design a service like Bitly that turns a long URL into a short code and redirects visitors to the original. This is the classic first system design problem because it forces you to reason about read-heavy traffic, unique key generation, and where a cache belongs — all without much business logic getting in the way. Your job here is to produce a design document, not a running service: decide how codes are generated, how the mapping is stored, and how a redirect stays fast when reads dwarf writes by a hundred to one.

## Prerequisites

- A mental model of how HTTP requests and 3xx redirects work
- Familiarity with the difference between a relational and a key-value store
- Having built the coding version helps ([URL Shortener (In-Memory)](../../../backend/beginner/02-url-shortener-in-memory/))
- Comfort reading a simple box-and-arrow diagram

## Learning Objectives

By the end, you should be able to:

- Separate functional requirements from non-functional ones (latency, availability)
- Do back-of-envelope math to size storage and QPS
- Compare short-code generation strategies and their collision behavior
- Justify where to place a cache in a read-heavy path
- Articulate one concrete trade-off in writing

## Requirements & Constraints

1. Shorten a valid `http`/`https` URL into a unique short code; reject invalid input.
2. Redirect a short code to its original URL with low latency (target p99 < 100 ms).
3. Reads vastly outnumber writes — assume a 100:1 read/write ratio.
4. Codes must be short (6–8 characters) and URL-safe.
5. The system should stay available for redirects even during write outages.
6. Estimate scale: ~10M new links/month, ~1B redirects/month.

## Suggested Approach

1. Write down the read path and the write path separately — they have different needs.
2. Do the estimation: 10M writes/month ≈ 4 writes/s; 1B reads/month ≈ 400 reads/s. Storage: 10M × ~500 bytes ≈ 5 GB/month.
3. Pick a code strategy — random base62 with a uniqueness check, or a counter encoded to base62.
4. Decide the datastore, then layer a cache in front of the read path.
5. Explain how a cache miss is handled and how the cache is populated.

## Architecture Sketch

```text
Client ── POST /shorten ──> [ App ] ──> [ Primary Store ]
Client ── GET /{code} ────> [ App ] ──> [ Cache ] --miss--> [ Store ]
                                             │
                                        302 Location: longUrl

Core API
  POST /shorten   { url }        -> 201 { code, shortUrl }
  GET  /{code}                   -> 302 Location: <longUrl> | 404

Data model
  links: code (PK) | long_url | created_at | hits
  counter (optional): single row/atomic sequence for base62 encoding
```

## Deep-Dive Topics

- **Key generation:** collision probability of random codes vs. the coordination cost of a global counter.
- **Caching:** LRU vs. TTL eviction; what cache hit rate you need to hit the latency target.
- **Availability:** why redirects can be served from a read replica or cache during a primary outage.

## Deliverables

- An architecture diagram showing the read and write paths.
- The core API contract (endpoints, methods, status codes).
- A data model for the link mapping.
- One key trade-off written up: e.g., random codes (no coordination, must retry on collision) vs. counter-based codes (predictable, but a coordination bottleneck).

## Common Pitfalls

- Designing the write path in detail and ignoring the read path that carries 99% of traffic.
- Choosing a code length without doing the math on how many URLs it can represent.
- Adding a cache without stating the eviction policy or how misses are handled.
- Treating "unique code" as free — every strategy has a collision or coordination cost.

## Resources

- [System Design Primer: Designing a URL shortener](https://github.com/donnemartin/system-design-primer#design-a-url-shortening-service-like-bitly) — a full worked example.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — the canonical open-source study guide.
- [MDN: HTTP Redirections](https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections) — 301 vs 302 and caching behavior.
- [Wikipedia: Base62](https://en.wikipedia.org/wiki/Base62) — the standard short-code encoding.
