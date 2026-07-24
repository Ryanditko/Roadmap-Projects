# Load Balancing System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a load balancer that sits in front of several backend servers and spreads incoming traffic across them, the role played by HAProxy, NGINX, or a cloud L4/L7 balancer. You will implement one or more distribution algorithms, actively health-check each backend so traffic never lands on a dead node, and drain a backend gracefully instead of cutting its in-flight requests. The subtle work is in the edges: keeping a client pinned to one backend when sessions require it, deciding fast enough that a failing node is pulled before users notice, and degrading sensibly when every backend is unhealthy. You come away understanding why a balancer is as much about health and failover as it is about "picking the next server".

## Prerequisites

- Comfort writing an HTTP or TCP proxy in your language of choice
- Understanding of connections, keep-alive, and request/response lifecycles
- Familiarity with health checks (from a supervisor or auto-scaler project)
- Basic grasp of hashing for consistent client-to-backend mapping
- A stepping stone: [Auto-Scaling Setup](../08-auto-scaling/) pairs naturally with this

## Learning Objectives

By the end, you should be able to:

- Implement and compare algorithms: round-robin, least-connections, and weighted variants
- Actively health-check backends and remove/restore them from the rotation automatically
- Support sticky sessions via cookie or consistent hashing when required
- Drain connections on removal so in-flight requests complete before a backend leaves
- Degrade gracefully — return a clear error, not a hang, when no backend is available

## Functional Requirements

1. The balancer must accept client requests and forward them to a healthy backend.
2. It must support at least round-robin and least-connections selection, chosen by config.
3. It must actively health-check each backend and exclude failing ones within a bounded time.
4. A recovered backend must rejoin the rotation automatically.
5. Sticky sessions must route a given client to the same backend while it stays healthy.
6. Removing a backend must drain existing connections rather than dropping them.
7. When all backends are unhealthy, the balancer must return a defined error, not hang or crash.

## Suggested Milestones

1. **Milestone 1 — Forward & rotate:** Proxy requests to a static pool using round-robin.
2. **Milestone 2 — Health & failover:** Add active health checks, exclusion, and automatic recovery.
3. **Milestone 3 — Stickiness & draining:** Add session affinity and graceful connection draining.

## Data & Interface Sketch

```text
backend
  id          string
  address     host:port
  weight      int
  status      up | down | draining
  in_flight   int          (for least-connections)

config
  algorithm     round_robin | least_conn | weighted
  health        { path, interval_s, timeout_s, unhealthy_after, healthy_after }
  sticky        none | cookie | ip_hash

selection:
  round_robin   next index mod pool
  least_conn    backend with min in_flight among up
  weighted      distribute proportional to weight

request path:
  client -> balancer -> pick backend (respect sticky) -> proxy
         backend down mid-request -> retry on another (idempotent only)
         no backend up -> 503 with clear body

health loop marks up/down after N consecutive results
```

## Stretch Goals

- Add connection pooling and keep-alive reuse to upstream backends.
- Add per-backend rate limiting and a simple circuit breaker for repeatedly failing nodes.
- Expose a metrics endpoint (requests, latency, per-backend health) and a small admin view.
- Support weighted canary routing to send a small percentage of traffic to a new backend.

## Definition of Done

- [ ] Traffic is distributed across backends according to the configured algorithm.
- [ ] A backend that fails its health check is removed within the configured window and later rejoins on recovery.
- [ ] Sticky sessions keep a client on one backend for the session's lifetime while it is healthy.
- [ ] Removing a backend drains in-flight requests instead of severing them.
- [ ] With every backend down, clients receive a defined error response, not a timeout.

## Common Pitfalls

- Health checks that only verify TCP connect, missing a backend that accepts connections but returns 500s.
- Retrying non-idempotent requests on failover, causing duplicate side effects.
- Sticky sessions with no fallback, so a client is stranded when its pinned backend dies.
- Removing a backend instantly on one failed check, causing flapping under transient blips.
- Cutting connections on drain, breaking long-running requests and uploads.

## Resources

- [HAProxy Documentation](https://docs.haproxy.org/) — algorithms, health checks, and stick tables in a real balancer.
- [NGINX: HTTP Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/) — practical configuration of the concepts here.
- [Cloudflare: What is Load Balancing?](https://www.cloudflare.com/learning/performance/what-is-load-balancing/) — a clear conceptual overview.
- [Google SRE Book: Load Balancing in the Datacenter](https://sre.google/sre-book/load-balancing-datacenter/) — health-aware balancing at scale.
