# Design a Rate Limiter

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Design a rate limiter that caps how many requests a client can make in a window — the component that returns `429 Too Many Requests` when someone hammers your API. The core question is which algorithm to use (token bucket, sliding window, fixed window) and where the counter lives. In a single server, counting is trivial; across a fleet behind a load balancer, the counter must be shared, which pulls in a fast central store and its consistency headaches. Deliver a design document covering the algorithm, the counter store, and the client contract.

## Prerequisites

- Understanding of what an API request and a time window are
- Awareness that multiple servers can't each keep their own private count
- Familiarity with a fast key-value store like Redis at a conceptual level
- Comfort reasoning about counters and expiry

## Learning Objectives

By the end, you should be able to:

- Compare rate-limiting algorithms and their burst behavior
- Decide where the counter lives for a distributed deployment
- Design the HTTP contract for a rejected request (status and headers)
- Reason about accuracy vs. performance in a shared counter
- State a trade-off between strict accuracy and low latency

## Requirements & Constraints

1. Limit each client (by API key or IP) to N requests per time window.
2. Reject over-limit requests with `429` and headers telling the client when to retry.
3. Work correctly across many application servers, not just one.
4. Add minimal latency to allowed requests (target overhead < 5 ms).
5. Support different limits per endpoint or per client tier.
6. Estimate scale: 50K requests/s peak across the fleet.

## Suggested Approach

1. Pick an algorithm and reason about its edge behavior — fixed window has boundary bursts; sliding window and token bucket smooth them.
2. Do the math: at 50K req/s, the counter store must handle at least that many read-modify-write ops — plan for it.
3. Decide the counter location: local (fast, inaccurate across nodes) vs. central store (accurate, adds a hop).
4. Define the response contract: `429` plus `Retry-After` and `X-RateLimit-*` headers.
5. Describe how counters expire and how per-tier limits are configured.

## Architecture Sketch

```text
Client ──> [ LB ] ──> [ App node ] ── check+incr ──> [ Central Counter Store ]
                             │                              (key: client:window)
                       allow │  deny
                             ▼    ▼
                          pass   429 + Retry-After

Algorithms
  Fixed window     count per fixed bucket           (simple, boundary bursts)
  Sliding window   weighted across two buckets       (smoother, more compute)
  Token bucket     refill tokens at a steady rate     (allows controlled bursts)

Data model (central store)
  key: "{clientId}:{window}"  value: count      TTL = window length
  config: client_id/tier -> limit, window
```

## Deep-Dive Topics

- **Algorithm choice:** why fixed-window allows a 2× burst at boundaries, and how token bucket permits intentional bursts.
- **Distributed counting:** atomic increment in the central store vs. approximate local counters synced periodically.
- **Failure mode:** fail-open (allow on store outage) vs. fail-closed (reject) and the risk of each.

## Deliverables

- An architecture diagram showing the app node, counter store, and decision path.
- The client contract: status code and rate-limit headers.
- A data model for the counter keys and per-tier config.
- One trade-off written up: e.g., central store (accurate global limits, extra latency and a dependency) vs. local counters (fast, but limits are only approximate across nodes).

## Common Pitfalls

- Keeping counts local to each server, so the real limit is N × (number of servers).
- Using a fixed window and ignoring the double-burst at window boundaries.
- Returning `429` with no `Retry-After`, leaving clients to guess and retry aggressively.
- Not deciding fail-open vs. fail-closed, so behavior during a store outage is undefined.

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — distributed systems and caching patterns.
- [Cloudflare: How we built rate limiting](https://blog.cloudflare.com/counting-things-a-lot-of-different-things/) — a production sliding-window approach.
- [Stripe: Scaling your API with rate limiters](https://stripe.com/blog/rate-limiters) — algorithms and practical trade-offs.
- [MDN: 429 Too Many Requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429) — the response and `Retry-After` header.
