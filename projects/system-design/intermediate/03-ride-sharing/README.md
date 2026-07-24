# Design a Ride-Sharing System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Design the backend for a ride-sharing platform like Uber or Lyft: riders request trips, the system finds the nearest available driver, both parties see each other move in real time, and payment settles when the trip ends. The core challenges are storing and querying millions of moving GPS points, matching supply to demand in milliseconds, and pricing dynamically when demand spikes. This is a design exercise: you produce a design document, capacity numbers, and diagrams — not working code.

## Prerequisites

- Understanding of geospatial indexing concepts (geohash, quadtree, S2/H3)
- Familiarity with real-time transport (WebSockets, long-lived connections)
- Awareness of sharding and hot-partition problems
- Comfort estimating write throughput from a stream of location pings

## Learning Objectives

By the end, you should be able to:

- Choose a geospatial index and explain nearest-driver queries
- Estimate location-update write QPS and storage for active drivers
- Design a matching service that avoids double-assigning a driver
- Partition location data geographically without creating hot cells
- Justify trade-offs between geohash precision and query breadth

## Requirements & Constraints

- Assume 1M active drivers, each pinging location every 4 s; ~5M riders/day.
- A nearest-driver query must return in under ~100 ms within a city.
- A driver must never be matched to two riders simultaneously.
- Location data is write-heavy and ephemeral; trip records are durable.
- Estimate location write QPS, geospatial index size, and trip-history storage/year.

## Suggested Approach

1. Compute location write QPS (drivers ÷ ping interval) and size the live index.
2. Pick a geospatial encoding (geohash/H3) and map cells to shards.
3. Design matching: query candidate drivers in nearby cells, then reserve one atomically.
4. Design the real-time channel for driver/rider position updates.
5. Add surge pricing driven by supply/demand ratio per cell.

## Architecture Sketch

```text
Driver app -- location ping (4s) --> [Location svc] -> Geo index (Redis GEO / H3 -> shard)
Rider app  -- request ride -------> [Matching svc] -> query nearby cells -> reserve driver (CAS)
                                          |-> [Trip svc] -> Trip DB (shard by tripId)
                                          |-> [Pricing svc] -> surge = f(demand/supply per cell)
Real-time positions <-- WebSocket gateway --> both apps

POST /rides            { riderId, pickup{lat,lng} } -> 202 { tripId, driverId, eta }
POST /location         { driverId, lat, lng, ts }   -> 204
GET  /trips/{tripId}                                -> 200 { status, route, fare }

Driver { driverId, cell, lat, lng, status, ts }   // key by cell, TTL on stale pings
Trip   { tripId, riderId, driverId, state, fare }  // shard by tripId
```

## Deep-Dive Topics

- **Geospatial indexing:** geohash vs. H3 cells; neighbor lookups at cell borders.
- **Matching integrity:** atomic reservation, timeouts, re-matching on driver decline.
- **Trade-off 1 — geohash precision:** fine cells give precise proximity but require querying many neighbor cells; coarse cells return too many candidates. Justify a precision tuned to city density.
- **Trade-off 2 — location store durability:** persisting every ping is expensive and rarely read; keep live positions in memory with TTL and only persist trip routes.

## Deliverables

- [ ] A design document (~3–5 pages) expanding the architecture above.
- [ ] Capacity estimates: location write QPS, geo index memory, trip-history storage/year.
- [ ] A geographic partitioning plan that addresses hot cells (e.g., downtown at rush hour).
- [ ] A caching/in-memory strategy for live positions with TTL policy.
- [ ] At least two trade-offs, each with the option chosen and why.

## Common Pitfalls

- Storing every GPS ping durably, exploding write cost for data no one queries.
- Ignoring cell-border cases so a driver one meter across a boundary is never matched.
- Non-atomic matching that assigns one driver to two riders under concurrency.
- Sharding purely by geography, so a stadium or airport becomes a permanent hot shard.

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — sharding and real-time patterns.
- [Uber Engineering: H3 hexagonal index](https://www.uber.com/blog/h3/) — geospatial partitioning in production.
- [Redis Geospatial commands](https://redis.io/docs/latest/develop/data-types/geospatial/) — proximity queries with GEO.
- [Martin Kleppmann, *Designing Data-Intensive Applications*](https://dataintensive.net/) — partitioning and hot spots.
