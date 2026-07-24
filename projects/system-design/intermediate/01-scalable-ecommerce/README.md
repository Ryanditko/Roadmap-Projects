# Design a Scalable E-commerce System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Design the backend for an online store like Amazon or Shopify: browsing a catalog, adding items to a cart, checking out, and tracking orders. The hard part is not the CRUD — it is keeping inventory consistent when thousands of buyers race for the last unit of a hot item, while keeping the catalog fast to browse. Browse traffic wants speed and can tolerate staleness; checkout wants correctness and cannot oversell. This is a design exercise: you produce a design document, capacity numbers, and diagrams, not working code.

## Prerequisites

- Familiarity with HTTP APIs and relational vs. NoSQL data models
- Basic understanding of database transactions and ACID
- Awareness of caching and message queues at a conceptual level
- Comfort reasoning about consistency vs. availability trade-offs

## Learning Objectives

By the end, you should be able to:

- Decompose a domain into services (catalog, cart, inventory, order, payment)
- Estimate read/write QPS and storage for a catalog and order stream
- Choose consistency models per subsystem — eventual for browse, strong for checkout
- Design an inventory reservation scheme that survives concurrent checkouts
- Justify trade-offs between a monolith and service decomposition

## Requirements & Constraints

- Assume 10M products, 5M daily active users, ~50k catalog reads/s peak, ~2k checkouts/s peak.
- Browsing must stay under ~200 ms p99; a stale price or stock count for a few seconds is acceptable.
- Checkout must never oversell: committed stock for an item cannot exceed physical stock.
- Payment capture and order creation must be atomic from the user's perspective.
- Estimate storage for the product catalog and one year of order history.

## Suggested Approach

1. Draw the request path for browse vs. checkout and note where consistency differs.
2. Size the catalog store and a read cache; compute the cache hit ratio needed to hit the QPS target.
3. Design inventory reservation: reserve-on-add-to-cart with TTL, or reserve-at-checkout.
4. Model checkout as a saga (reserve → pay → confirm) with compensating actions on failure.
5. Pick partitioning keys for catalog, cart, and orders and defend them.

## Architecture Sketch

```text
Clients -> CDN/Edge -> API Gateway -> [Catalog svc] -> Catalog DB (read replicas) + Redis cache
                                    -> [Cart svc]    -> Cart store (Redis/Dynamo, user-keyed)
                                    -> [Order svc]   -> Order DB (sharded by order_id)
                                          |-> Inventory svc -> Inventory DB (row lock / CAS)
                                          |-> Payment svc   -> external PSP
                          Events -> Kafka -> [search indexer, analytics, email]

POST /cart/{userId}/items      { sku, qty } -> 200 cart
POST /checkout                 { userId }   -> 202 { orderId, status: PENDING }
GET  /orders/{orderId}                      -> 200 { status, items, total }

Product   { sku, title, price, attrs{}, categoryId }      // partition by sku hash
Inventory { sku, available, reserved, version }           // optimistic concurrency
Order     { orderId, userId, items[], total, status, ts } // shard by orderId
```

## Deep-Dive Topics

- **Inventory consistency:** reservation TTLs, optimistic vs. pessimistic locking, oversell recovery.
- **Checkout saga:** orchestration vs. choreography; idempotency keys on payment.
- **Trade-off 1 — monolith vs. services:** faster to ship vs. independent scaling of catalog reads. Justify splitting read-heavy catalog from write-heavy orders.
- **Trade-off 2 — cache freshness:** aggressive caching cuts DB load but risks selling at a stale price; use short TTLs plus event-driven invalidation.

## Deliverables

- [ ] A design document (~3–5 pages) expanding the architecture diagram above.
- [ ] Capacity estimates: catalog QPS, checkout QPS, cache hit ratio, catalog + order storage.
- [ ] A partitioning plan for catalog, cart, and orders with rationale.
- [ ] A caching strategy with an invalidation approach.
- [ ] At least two trade-offs, each with the option chosen and why.

## Common Pitfalls

- Applying one consistency model everywhere — browse and checkout have different needs.
- Reserving inventory without a TTL, so abandoned carts leak stock forever.
- Making payment non-idempotent, so a retried checkout double-charges.
- Forgetting to size the cache; a low hit ratio quietly pushes all load onto the DB.

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — capacity estimation and scaling patterns.
- [Amazon Builders' Library: Using load shedding to avoid overload](https://aws.amazon.com/builders-library/using-load-shedding-to-avoid-overload/) — protecting checkout under load.
- [Martin Kleppmann, *Designing Data-Intensive Applications*](https://dataintensive.net/) — consistency and partitioning.
- [Saga pattern (Microservices.io)](https://microservices.io/patterns/data/saga.html) — distributed checkout transactions.
