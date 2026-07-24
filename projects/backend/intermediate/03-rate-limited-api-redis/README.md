# Rate-Limited API with Redis

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Every public API eventually needs a bouncer at the door — something that lets regular traffic through but stops one noisy client from swamping the service. In this project you build that bouncer as reusable rate-limiting middleware backed by Redis. The interesting twist is that your API might run as several instances behind a load balancer, so counting requests in local memory would let a client multiply its allowance by the number of servers. Redis becomes the single source of truth that every instance shares, and you get to weigh real algorithms — fixed window, sliding window, token bucket — against each other instead of reaching for a library that hides the trade-offs.

## Prerequisites

- Comfort building HTTP endpoints and middleware ([Simple REST API for Task Management](../../beginner/01-simple-rest-api-task-management/) is a good base)
- A running Redis instance (local Docker container is fine)
- Understanding of HTTP status codes, especially `429 Too Many Requests`
- Familiarity with atomic operations and why they matter under concurrency

## Learning Objectives

By the end, you should be able to:

- Compare fixed-window, sliding-window, and token-bucket algorithms and their memory/accuracy trade-offs
- Use Redis atomic commands (`INCR`, `EXPIRE`, sorted sets, or a Lua script) to count safely across instances
- Return the standard rate-limit headers so clients can self-throttle
- Distinguish between per-user (authenticated) and per-IP (anonymous) limiting
- Reason about what happens when Redis is slow or unavailable

## Functional Requirements

1. The system must enforce a configurable limit of N requests per time window on each protected endpoint.
2. Requests over the limit must be rejected with HTTP `429` and a `Retry-After` header.
3. Counting must be shared across multiple API instances via Redis, not held in local memory.
4. The counter increment and its expiry must be atomic so concurrent requests cannot bypass the limit.
5. Every response must include `RateLimit-Limit`, `RateLimit-Remaining`, and `RateLimit-Reset` headers.
6. Authenticated requests must be limited per user identity; anonymous requests per client IP.
7. A configured allowlist of identities or IPs must bypass limiting entirely.

## Suggested Milestones

1. **Milestone 1 — Fixed window:** Implement `INCR` + `EXPIRE` per key/window, returning 429 past the limit.
2. **Milestone 2 — Better algorithm:** Replace the fixed window with a sliding window (sorted set) or token bucket to smooth burst behavior at window edges.
3. **Milestone 3 — Headers, identities & allowlist:** Emit the standard headers, split per-user vs per-IP keys, and honor an allowlist.

## Data & Interface Sketch

```text
Redis keys
  ratelimit:user:{id}:{window}   -> counter (fixed window)
  ratelimit:ip:{addr}            -> sorted set of request timestamps (sliding)
  tokenbucket:{id}               -> hash { tokens, lastRefill }

Response headers (allowed)
  RateLimit-Limit: 100
  RateLimit-Remaining: 87
  RateLimit-Reset: 1710000000

Response when blocked
  HTTP 429 Too Many Requests
  Retry-After: 15
  body: { "error": "rate_limit_exceeded", "retryAfter": 15 }

Algorithm choices: fixed window | sliding window log | token bucket
```

## Stretch Goals

- Add tiered limits (free vs premium plans) resolved from the authenticated identity.
- Implement burst allowance on top of a steady refill rate with the token-bucket model.
- Add an admin endpoint to inspect and adjust limits at runtime.
- Move the check-and-increment logic into a single Redis Lua script to eliminate round trips.

## Definition of Done

- [ ] A client exceeding the limit consistently receives 429 with a correct `Retry-After`.
- [ ] Running two API instances against one Redis enforces a single shared limit.
- [ ] Concurrent requests near the boundary never exceed the configured count.
- [ ] Standard rate-limit headers appear on both allowed and blocked responses.
- [ ] Allowlisted identities are never throttled.

## Common Pitfalls

- Using separate `GET` then `SET` instead of an atomic `INCR` — a race lets requests slip through.
- Forgetting to set an expiry on the first increment, so counters live forever and never reset.
- The fixed-window edge problem: a client can send 2×N requests across the boundary of two windows.
- Blocking every request on a synchronous Redis call with no timeout, so a slow Redis stalls the whole API.
- Keying anonymous limits on an IP behind a proxy without reading `X-Forwarded-For`, lumping all users together.

## Resources

- [Redis: INCR command](https://redis.io/docs/latest/commands/incr/) — the canonical atomic-counter pattern, with a rate-limiter example.
- [Redis: rate limiting patterns](https://redis.io/glossary/rate-limiting/) — sliding window and token bucket explained.
- [MDN: 429 Too Many Requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429) — status semantics and `Retry-After`.
- [IETF draft: RateLimit header fields](https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/) — the standard headers your API should emit.
- [Cloudflare: how rate limiting works](https://www.cloudflare.com/learning/bots/what-is-rate-limiting/) — a plain-language overview of the concepts.
