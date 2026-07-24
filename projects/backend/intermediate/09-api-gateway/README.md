# API Gateway (basic routing)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build the single front door that sits in front of several backend services and decides where each request goes. Clients talk only to the gateway; it inspects the path, picks the right upstream, forwards the request, and streams the response back. Along the way it does the cross-cutting work no individual service should repeat — validating the JWT once, enforcing timeouts, retrying transient failures, and tripping a circuit breaker when an upstream is sick. This is how tools like Kong, NGINX, and AWS API Gateway earn their keep, and building a small one teaches you where latency, failure, and trust boundaries actually live in a distributed system.

## Prerequisites

- Comfort writing HTTP servers and making outbound HTTP calls from your language of choice
- Understanding of JWT verification (a warm-up is [E-commerce API with JWT](../01-ecommerce-api-jwt/))
- Familiarity with HTTP status codes, headers, and the request/response lifecycle
- Two or three tiny backend services (or stubs) to route to
- Basic grasp of concurrency and connection reuse

## Learning Objectives

By the end, you should be able to:

- Route incoming requests to the correct upstream using path-based rules
- Validate authentication centrally so upstreams can trust the gateway
- Distribute load across healthy replicas with round-robin balancing
- Apply per-upstream timeouts and bounded retries without amplifying failure
- Implement a circuit breaker that isolates a failing service and recovers automatically
- Distinguish gateway-originated errors (502/503/504) from upstream ones

## Functional Requirements

1. The gateway must forward requests to a backend chosen by matching the request path against a service registry.
2. It must return 404 when no route matches and 502/503 when the chosen upstream is unreachable.
3. It must validate the `Authorization` bearer token once and reject invalid tokens with 401 before forwarding.
4. It must load-balance across multiple instances of the same service using round-robin.
5. It must enforce a configurable per-request timeout and return 504 when an upstream exceeds it.
6. It must retry idempotent requests a bounded number of times on transient (5xx/connection) failures.
7. It must open a circuit breaker after repeated upstream failures and fast-fail until a half-open probe succeeds.
8. It must forward the original method, headers, query string, and body, and preserve the upstream status and body on the way back.

## Suggested Milestones

1. **Milestone 1 — Reverse proxy:** Match a path to one upstream and faithfully forward the request and response.
2. **Milestone 2 — Registry & balancing:** Support multiple services with multiple instances and round-robin selection.
3. **Milestone 3 — Auth & resilience:** Add central JWT validation, timeouts, bounded retries, and a circuit breaker with health checks.

## Data & Interface Sketch

```text
Service registry (config)
  services:
    - name: "orders"
      prefix: "/orders"
      instances: ["http://orders-1:8080", "http://orders-2:8080"]
      timeoutMs: 2000
      auth: true

Request flow
  client -> gateway
    match prefix -> service
    if service.auth: verify JWT (401 on failure)
    pick instance (round-robin, skip open circuits)
    forward with timeout -> upstream
    on 5xx/timeout: retry (idempotent only, max N)
    trip breaker after K consecutive failures

Breaker states: CLOSED -> OPEN -> HALF_OPEN -> CLOSED
Gateway errors: 404 no route | 401 bad token | 502 bad upstream
                503 circuit open | 504 timeout
```

## Stretch Goals

- Add response aggregation: one gateway route that fans out to two upstreams and merges results.
- Support request/response header transformation (inject `X-Request-Id`, strip hop-by-hop headers).
- Add weighted load balancing so a canary instance receives a small percentage of traffic.
- Expose a metrics endpoint reporting per-upstream latency, error rate, and breaker state.

## Definition of Done

- [ ] A request to a known prefix reaches the right upstream and the response is returned unchanged.
- [ ] Unknown paths return 404 and unreachable upstreams return 502/503, never a hang.
- [ ] An invalid or missing token is rejected with 401 before any upstream call is made.
- [ ] Traffic spreads evenly across healthy instances, and a downed instance is skipped.
- [ ] A slow upstream trips the timeout (504); repeated failures open the breaker and it later recovers.

## Common Pitfalls

- Forwarding hop-by-hop headers (`Connection`, `Transfer-Encoding`) verbatim, corrupting the proxied response.
- Retrying non-idempotent requests (POST payments), causing duplicate side effects.
- No timeout on the outbound call, so one slow upstream exhausts the gateway's connection pool.
- Treating a 401 from the upstream as a gateway error, hiding the real cause from the client.
- A circuit breaker that opens but never probes, permanently blackholing a recovered service.

## Resources

- [NGINX: What Is an API Gateway?](https://www.nginx.com/learn/api-gateway/) — the pattern and its responsibilities.
- [Martin Fowler: CircuitBreaker](https://martinfowler.com/bliki/CircuitBreaker.html) — the canonical explanation of the states and transitions.
- [MDN: Proxy servers and tunneling](https://developer.mozilla.org/en-US/docs/Web/HTTP/Proxy_servers_and_tunneling) — hop-by-hop vs end-to-end headers.
- [roadmap.sh: Backend Developer](https://roadmap.sh/backend) — where gateways fit in the wider backend picture.
