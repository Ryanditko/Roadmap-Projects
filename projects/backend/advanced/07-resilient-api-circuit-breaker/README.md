# Resilient API with Circuit Breaker

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build an API that stays useful when its dependencies do not. Real systems call other systems — a payment gateway, a recommendation service, a database — and any of those can slow down or fail. Left unguarded, one slow dependency ties up every request thread and takes the whole service down with it. This project is about the resilience patterns that stop that cascade: the **circuit breaker** that stops hammering a failing dependency, the **bulkhead** that isolates one dependency's failures from the rest, **timeouts** that cap how long you wait, **fallbacks** that return something sensible instead of an error, and **load shedding** that protects the service under overload. The goal is graceful degradation — a service that gets narrower under stress instead of falling over.

## Prerequisites

- Solid experience building HTTP APIs and calling downstream services
- Understanding of threads, connection pools, and blocking vs. async I/O
- Comfort with concurrency primitives (thread pools, semaphores, timeouts)
- Familiarity with basic metrics/monitoring concepts
- A backend stack of your choice (Java/Kotlin, Go, Node, Python, etc.)

## Learning Objectives

By the end, you should be able to:

- Implement a circuit breaker as a state machine (closed → open → half-open)
- Isolate dependencies with bulkheads so one failure doesn't exhaust shared resources
- Choose and enforce sensible timeouts, and explain why "no timeout" is a bug
- Design fallbacks that degrade functionality gracefully instead of erroring
- Shed load under overload to protect latency for the requests you do serve
- Reason about the trade-offs of each pattern (freshness, false trips, tuning)

## Functional Requirements

1. The service must wrap calls to an unreliable downstream dependency behind a circuit breaker with three states: **closed** (calls pass), **open** (calls fail fast), and **half-open** (a limited probe of calls tests recovery).
2. The breaker must open when a configurable failure threshold is crossed (error rate or consecutive failures) and, after a cooldown, move to half-open to test the dependency.
3. Every downstream call must have a timeout; a call exceeding it counts as a failure, never an indefinite wait.
4. Each downstream dependency must be isolated by a bulkhead (a bounded thread pool or semaphore) so saturating one cannot starve the others.
5. When the breaker is open or a call fails, the service must return a defined fallback (cached value, default, or partial response) rather than propagating a raw error where possible.
6. The service must shed load when it is past a concurrency or queue-depth limit, rejecting excess requests quickly with 503 instead of queueing them indefinitely.
7. **Non-functional:** under a fully-down dependency, p99 latency of the protected endpoint must stay bounded (fail-fast), and the service must not exhaust its own threads or connections. It must remain available for endpoints that don't depend on the failed service.
8. The service must expose metrics (breaker state, call success/failure counts, rejected/shed requests, latency) for observability.

## Suggested Milestones

1. **Milestone 1 — Timeouts & a wrapped call:** Put a real (or simulated flaky) dependency behind a client with an enforced timeout and structured success/failure results.
2. **Milestone 2 — Circuit breaker:** Implement the closed/open/half-open state machine with threshold and cooldown; verify it fails fast when open.
3. **Milestone 3 — Bulkhead & fallback:** Add per-dependency isolation and a fallback path; prove one dependency's failure doesn't affect another.
4. **Milestone 4 — Load shedding & metrics:** Add a concurrency limit with fast rejection and expose breaker/latency metrics; run a load test to observe degradation.

## Data & Interface Sketch

```text
Circuit Breaker state machine

        failures >= threshold
 CLOSED ─────────────────────────▶ OPEN
   ▲                                 │
   │ probe succeeds                  │ cooldown elapsed
   │                                 ▼
   └────────── HALF-OPEN ◀───────────┘
        probe fails → back to OPEN

Request path
 client ─▶ [load shedder] ─▶ [bulkhead pool] ─▶ [breaker] ─▶ downstream
                 │                                   │
                 └─ 503 if over limit                └─ fallback if open/failed

Config (per dependency)
  timeoutMs, failureThreshold, cooldownMs,
  bulkheadMaxConcurrent, halfOpenProbes

Metrics exposed
  breaker_state{dep}          gauge (0=closed,1=open,2=half_open)
  calls_total{dep,result}     counter
  requests_shed_total         counter
  call_latency_seconds        histogram
```

## Stretch Goals

- Add **adaptive timeouts** based on observed latency percentiles instead of a fixed value.
- Implement a **retry with exponential backoff and jitter** in front of the breaker, and reason about how retries and breakers interact.
- Add a small **dashboard** or status endpoint visualizing live breaker states.
- Support **priority-based load shedding**, keeping critical requests while dropping low-priority ones.

## Definition of Done

- [ ] The breaker demonstrably opens under sustained failures and recovers via half-open probes.
- [ ] A dead dependency causes fast failures, not thread exhaustion or unbounded latency.
- [ ] Bulkheads prevent one saturated dependency from degrading unrelated endpoints.
- [ ] Fallbacks return usable responses where defined, with clear behavior where not.
- [ ] Load shedding rejects excess traffic quickly and keeps served-request latency stable.
- [ ] Metrics expose breaker state, call outcomes, and shed counts.

## Common Pitfalls

- Making downstream calls with no timeout — the single most common cause of cascading failure.
- Retrying aggressively in front of an open breaker, amplifying load on a struggling dependency.
- Sharing one thread pool across all dependencies, so one slow call starves everything (no real bulkhead).
- Setting the failure threshold so low the breaker trips on normal transient blips (false positives).
- Fallbacks that quietly serve stale or wrong data without any signal that degradation occurred.
- Load-shedding by letting requests queue until they time out anyway, instead of rejecting fast.

## Resources

- [Martin Fowler: CircuitBreaker](https://martinfowler.com/bliki/CircuitBreaker.html) — the canonical explanation of the pattern.
- [Release It! (Michael Nygard)](https://pragprog.com/titles/mnee2/release-it-second-edition/) — the book that popularized circuit breakers, bulkheads, and timeouts.
- [Resilience4j documentation](https://resilience4j.readme.io/docs/getting-started) — a reference implementation of these patterns to study.
- [AWS: Timeouts, retries, and backoff with jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/) — how retries and backoff interact with resilience.
- [Google SRE Book: Handling Overload](https://sre.google/sre-book/handling-overload/) — load shedding and graceful degradation in practice.
