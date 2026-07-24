# Design an Uber-like Ride-Sharing Platform

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Advanced · **Estimated time:** 3–7 days

## Overview

Design a ride-hailing platform in the style of Uber: riders request a trip, the system finds a nearby available driver in seconds, tracks both parties in real time, and settles payment when the ride ends. The hard part is the marriage of two continuous streams — a firehose of driver GPS pings and a burst of rider requests — resolved by a geo-spatial matching engine under a tight latency budget, all while surge pricing rebalances supply and demand. This is a design exercise: your deliverable is a written design document with diagrams and trade-off analysis, not running code.

## Prerequisites

- Comfort with real-time systems (WebSockets, pub/sub, streaming)
- Familiarity with geo-spatial indexing concepts (geohash, quadtrees, S2 cells)
- Understanding of eventual consistency and idempotent writes
- Basic grasp of payment flows and distributed transactions

## Learning Objectives

By the end, you should be able to:

- Design a geo-spatial index that answers "nearest available drivers" at city scale
- Model the driver-location firehose and choose write/read paths that survive its volume
- Design a matching algorithm that balances rider wait time against driver utilization
- Reason about surge pricing as a supply/demand feedback loop
- Handle payments with exactly-once semantics despite retries and failures

## Requirements & Constraints

1. Match a rider to a driver within a few seconds, p99 under ~2 s in dense areas.
2. Ingest driver location updates every ~4 s from millions of active drivers.
3. Guarantee a driver is offered to at most one rider at a time (no double-dispatch).
4. Compute fares deterministically and charge exactly once per completed trip.
5. Apply surge multipliers per geographic cell, updated on a short cadence.
6. Target 99.95% availability; a failed match must degrade gracefully with retry.
7. Enforce per-region compliance (pricing rules, driver vetting, data residency).

## Suggested Approach

Estimate the location-write throughput first (drivers × ping frequency) — it dwarfs read traffic and shapes your storage choice. Partition the world into geo cells (e.g. S2/geohash) and keep a hot in-memory index of driver positions per cell, rebuilt from a stream. Model matching as: locate candidate cells → query nearby drivers → rank by ETA → offer with a short lock/TTL. Treat surge as a separate service that reads demand/supply per cell and publishes multipliers. Keep the trip lifecycle as a state machine backed by durable events, and isolate payments behind an idempotency key so retries never double-charge.

## Architecture Sketch

```text
Driver app ──ping──> Location ingest (stream) ──> Geo index (in-mem per cell) 
                                                        ^
Rider app ──request──> Trip svc ──match──> Matching svc ┘
                          │
                          ├──> Pricing/Surge svc ──> demand/supply per cell
                          └──> Payments svc ──(idempotency key)──> ledger + PSP

Trip lifecycle (state machine):
REQUESTED -> MATCHED -> ACCEPTED -> IN_PROGRESS -> COMPLETED -> PAID
                 └────────> TIMED_OUT/CANCELLED

Key APIs:
POST /trips                 -> { tripId, status: REQUESTED }
POST /drivers/{id}/location -> 202 (fire-and-forget, batched)
POST /trips/{id}/accept     -> { status: ACCEPTED }  (driver-side)
GET  /trips/{id}            -> { status, driverLoc, etaSeconds }

Data model (sketch):
Driver{ id, cellId, status: AVAILABLE|BUSY, lastPingAt }
Trip{ id, riderId, driverId, state, fare, surgeMultiplier }
```

## Deep-Dive Topics

- **Geo-spatial indexing:** S2 vs geohash vs quadtree; cell size vs query fan-out trade-off.
- **Matching under contention:** dispatch locks, TTLs, avoiding double-offer, backoff on decline.
- **Surge pricing loop:** measuring demand/supply, damping oscillation, fairness and caps.
- **Payment idempotency:** exactly-once charging, saga/compensation for failed captures.
- **Hotspot mitigation:** stadium-let-out spikes concentrated in a few cells.

## Deliverables

- [ ] A design document (~4–8 pages) with the architecture and trip state machine, refined.
- [ ] Capacity estimates for location writes, geo-index memory, and match QPS at peak.
- [ ] The matching algorithm described with its consistency and locking strategy.
- [ ] A failure/DR analysis: geo-index node loss, payment PSP outage, region isolation.
- [ ] Hotspot mitigation plan for demand spikes concentrated in small areas.

## Common Pitfalls

- Storing every GPS ping in a strongly consistent DB — the write volume will crush it; stream it.
- Offering one driver to multiple riders because dispatch has no lock/TTL.
- Charging twice on retry because the payment path lacks an idempotency key.
- Global surge instead of per-cell, so pricing swings wildly and unfairly.
- Choosing geo cells too large (poor matches) or too small (huge query fan-out).

## Resources

- [Uber Engineering Blog](https://www.uber.com/en/blog/engineering/) — real accounts of matching and geo systems.
- [S2 Geometry](https://s2geometry.io/) — the spherical cell library used for geo-indexing.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — sharding, caching, and real-time patterns.
- [Designing Data-Intensive Applications](https://dataintensive.net/) — consistency, streams, and idempotency in depth.
- [Stripe: Idempotent requests](https://docs.stripe.com/api/idempotent_requests) — the canonical exactly-once payment pattern.
