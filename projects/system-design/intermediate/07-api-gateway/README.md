# Design an API Gateway

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Design an API gateway that sits in front of a microservices fleet: it is the single entry point that authenticates callers, routes each request to the right backend, enforces rate limits, and shields services from traffic spikes. Every request in the system passes through it, so it must be fast, highly available, and stateless enough to scale horizontally. The design questions are where to keep shared state (auth tokens, rate-limit counters), how to discover backends, and how to fail gracefully. This is a design exercise: you produce a design document, capacity numbers, and diagrams — not working code.

## Prerequisites

- Understanding of reverse proxies and load balancing
- Familiarity with token-based auth (JWT/OAuth) at a conceptual level
- Awareness of rate-limiting algorithms (token bucket, sliding window)
- Comfort estimating request throughput and per-request latency budget

## Learning Objectives

By the end, you should be able to:

- Design the request pipeline: auth → rate limit → route → transform → forward
- Estimate gateway QPS and the added latency budget per stage
- Design a distributed rate limiter with shared counters
- Design service discovery and health checking for dynamic backends
- Justify trade-offs between centralized and per-instance state

## Requirements & Constraints

- Assume 100k requests/s across 50 backend services, p99 added latency under ~10 ms.
- The gateway must remain available even if some backends are down (circuit breaking).
- Rate limits are enforced per API key across the whole gateway fleet, not per instance.
- Auth must validate tokens without a round-trip per request where possible.
- Estimate gateway QPS, rate-limit counter storage, and cache size for auth/routes.

## Suggested Approach

1. Define the request pipeline stages and the latency budget for each.
2. Decide where auth validation happens: local JWT verification vs. calling an auth service.
3. Design distributed rate limiting: shared counters in Redis vs. approximate local counters.
4. Design service discovery (registry + health checks) and load balancing.
5. Add resilience: circuit breakers, timeouts, retries with backoff.

## Architecture Sketch

```text
Clients -> [LB] -> Gateway fleet (stateless) --pipeline--> backends
                       |  1. auth (verify JWT locally; cache JWKS)
                       |  2. rate limit (Redis token bucket per apiKey)
                       |  3. route (path -> service via registry)
                       |  4. transform (headers, versioning)
                       |  5. forward (+ circuit breaker, timeout, retry)
        Service registry <- health checks -> backend instances

ANY /v1/{service}/*   -> forward to resolved backend
GET /_health          -> 200 gateway health

Route      { pathPrefix, serviceId, version }              // cached in-memory, refreshed
RateLimit  { apiKey -> tokens, refillTs }                  // Redis, partition by apiKey
Backend    { serviceId, instances[{host, healthy}], lbPolicy }
```

## Deep-Dive Topics

- **Rate limiting:** token bucket vs. sliding window; shared counter consistency vs. speed.
- **Resilience:** circuit breaker states (closed/open/half-open), timeout and retry budgets.
- **Trade-off 1 — centralized vs. local rate-limit state:** a shared Redis counter is exact but adds a network hop and a dependency; local counters are fast but only approximate the global limit. Justify local counters with periodic sync for high QPS.
- **Trade-off 2 — auth strategy:** local JWT verification avoids a per-request round-trip but makes token revocation slow; a central auth call is authoritative but adds latency. Justify short-lived JWTs plus a revocation cache.

## Deliverables

- [ ] A design document (~3–5 pages) expanding the pipeline and architecture above.
- [ ] Capacity estimates: gateway QPS, per-stage latency budget, rate-limit + route cache storage.
- [ ] A plan for partitioning rate-limit counters by API key.
- [ ] A caching strategy for JWKS, routes, and auth decisions with refresh policy.
- [ ] At least two trade-offs, each with the option chosen and why.

## Common Pitfalls

- Making the gateway stateful, so instances cannot scale or fail over cleanly.
- Enforcing rate limits per instance, letting total traffic exceed the intended global limit.
- No circuit breaker, so one slow backend exhausts gateway threads and takes down everything.
- Validating tokens by calling the auth service on every request, blowing the latency budget.

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — proxies, load balancing, and reliability.
- [NGINX: what is an API gateway](https://www.nginx.com/learn/api-gateway/) — gateway responsibilities in practice.
- [Martin Fowler: Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html) — the resilience pattern explained.
- [Stripe: scaling your API with rate limiters](https://stripe.com/blog/rate-limiters) — real-world distributed rate limiting.
