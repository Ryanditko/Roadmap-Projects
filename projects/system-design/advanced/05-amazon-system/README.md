# Design an Amazon-like E-Commerce Platform

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Advanced · **Estimated time:** 3–7 days

## Overview

Design a large-scale e-commerce platform in the style of Amazon: a catalog of hundreds of millions of products, full-text search with relevance ranking, a shopping cart that survives sessions, an inventory system that never oversells, and an order pipeline that stays correct under Black Friday load. The design tension is between a read-optimized browsing experience and a strongly-consistent checkout where money and stock counts must be exact. This is a design exercise: your deliverable is a written design document with diagrams and trade-off analysis, not running code.

## Prerequisites

- Solid understanding of relational vs NoSQL data models and when each fits
- Familiarity with search engines (inverted index, relevance ranking)
- Understanding of transactions, isolation levels, and idempotency
- Exposure to event-driven / saga patterns for multi-step workflows

## Learning Objectives

By the end, you should be able to:

- Partition a catalog and search index across shards while keeping queries fast
- Design inventory reservation that prevents overselling under concurrency
- Model an order as a saga across cart, payment, inventory, and fulfillment
- Choose consistency per domain: eventual for browse, strong for checkout/inventory
- Plan for peak-event (Black Friday) traffic that is 10–50× baseline

## Requirements & Constraints

1. Serve product pages and search with p99 latency under ~300 ms at steady state.
2. Never oversell: a unit reserved for one order cannot be sold to another.
3. Process orders exactly once even when clients and services retry.
4. Support ~300M products and peak traffic of tens of millions of requests/minute.
5. Keep cart contents durable across devices and sessions.
6. Isolate checkout consistency from browse availability (browse degrades, checkout stays correct).
7. Support third-party sellers with independent inventory and pricing.

## Suggested Approach

Split read and write paths hard. The browse/search plane is read-optimized: denormalized product documents in a search index plus a cache, tolerant of eventual consistency. The checkout plane is transactional: reserve inventory with a conditional write (optimistic concurrency or a reservation record with TTL), then run the order as a saga with compensating actions. Estimate peak throughput and design the reservation store to be the scaling bottleneck you protect. Use idempotency keys on order submission. For Black Friday, precompute hot product pages, shed non-critical load, and queue writes so the inventory store degrades gracefully rather than oversells.

## Architecture Sketch

```text
Browse: client ──> API ──> Search svc (index shards) + Product cache ──> Catalog DB (source of truth)
Cart:   client ──> Cart svc ──> durable cart store (per user)

Checkout (saga):
  submit(order, idempotencyKey)
    -> Inventory svc: reserve (conditional/TTL)  --fail--> reject
    -> Payment svc: authorize (idempotent)        --fail--> release reservation
    -> Order svc: create ORDER (PLACED)
    -> Fulfillment svc: async pick/pack/ship
  compensation on any failure rolls back prior steps

Key APIs:
GET  /search?q=...&filters=...     -> { results[], facets }
POST /cart/items                   -> { cart }
POST /orders (Idempotency-Key)     -> { orderId, status: PLACED }

Data model (sketch):
Product{ id, sellerId, attrs, priceCents }            # eventual for browse
Inventory{ sku, available, reserved }                 # strong, conditional writes
Order{ id, userId, items[], state, idempotencyKey }   # saga-driven
```

## Deep-Dive Topics

- **Inventory concurrency:** reservations, optimistic vs pessimistic locking, TTL cleanup.
- **Order saga:** compensating transactions, exactly-once via idempotency keys.
- **Search sharding & relevance:** index partitioning, ranking signals, faceting.
- **Consistency split:** eventual browse cache vs strongly-consistent checkout.
- **Peak-event scaling:** load shedding, write queuing, hot-page precomputation.

## Deliverables

- [ ] A design document (~4–8 pages) with browse and checkout planes separated, refined.
- [ ] Capacity estimates for catalog size, search QPS, and peak checkout throughput.
- [ ] The inventory-reservation and order-saga mechanisms fully specified.
- [ ] A failure/DR analysis: payment outage mid-saga, inventory-store partition, cache stampede.
- [ ] A Black Friday plan: what degrades, what stays strongly consistent, and why.

## Common Pitfalls

- Decrementing stock only at payment time, allowing two orders to oversell the last unit.
- Making the whole checkout one giant distributed transaction instead of a compensable saga.
- Serving checkout from the eventually-consistent browse cache, showing stale prices/stock.
- No idempotency key on order submit, so a retry creates duplicate orders and charges.
- Treating Black Friday as "just more traffic" without load shedding or write queuing.

## Resources

- [Dynamo: Amazon's Highly Available Key-value Store](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) — the canonical paper on availability and eventual consistency at Amazon.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — sharding, caching, and consistency patterns.
- [Saga pattern (microservices.io)](https://microservices.io/patterns/data/saga.html) — compensating transactions for multi-service workflows.
- [Designing Data-Intensive Applications](https://dataintensive.net/) — isolation levels and distributed transactions.
- [Amazon Builders' Library](https://aws.amazon.com/builders-library/) — real reliability and scaling practices.
