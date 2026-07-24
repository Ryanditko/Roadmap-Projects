# Data Mesh Simulation

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Simulate a data mesh: instead of one central team owning a monolithic warehouse, several independent domains each publish their data as a **data product** with an owner, a versioned contract, quality SLOs, and a discoverable entry in a catalog. You will model two or three domains (say, Orders, Payments, and Customers), give each a published output port, and enforce a **data contract** so that a breaking schema change is caught before it reaches consumers. The hard part is organizational made technical: how do you keep domains autonomous while still guaranteeing a consumer can join across them? You will design federated governance — global rules everyone follows, local freedom in everything else — and prove it with a cross-domain query that only works because contracts held.

## Prerequisites

- Experience building data pipelines and thinking about producers vs consumers
- Familiarity with schema formats and evolution (Avro, Protobuf, or JSON Schema)
- Understanding of a data catalog / metadata store concept
- Comfort with the tradeoffs of decentralized ownership (this is an architecture exercise as much as a coding one)

## Learning Objectives

By the end, you should be able to:

- Model a domain as a self-contained data product with a clear output port
- Define and version a data contract, and detect breaking vs compatible changes
- Design federated governance: which rules are global, which are local
- Make data products discoverable through a catalog with ownership and SLOs
- Reason about cross-domain consistency without a single owning team

## Functional Requirements

1. Each domain must expose at least one data product with a documented schema, owner, and freshness/quality SLO.
2. Every data product must be registered in a shared catalog that a consumer can search by domain, owner, or field.
3. A published contract must be validated on every new dataset version; a backward-incompatible change must be rejected or flagged before publish.
4. A consumer must be able to run a query that joins two domains' products using only their public contracts.
5. Global governance rules (e.g. every product carries a `data_owner` and a PII classification) must be enforced uniformly.
6. Contract violations and SLO breaches must surface to the owning domain, not to a central team.

## Suggested Milestones

1. **Milestone 1 — Domains & products:** Model 2–3 domains, each publishing a data product with schema, owner, and SLO metadata.
2. **Milestone 2 — Contracts & catalog:** Add contract validation on publish and register everything in a searchable catalog.
3. **Milestone 3 — Federated governance:** Enforce global rules, wire SLO/contract alerts to domain owners, and demo a cross-domain join.

## Data & Interface Sketch

```text
Domain "orders"                    Domain "payments"
  data product: orders_daily         data product: settlements_daily
  owner: orders-team@example.com     owner: payments-team@example.com
  contract v2 { orderId, amount,     contract v1 { orderId, settledAt,
    currency, dt }  slo: freshness<2h  status }  slo: completeness>99.9%
        │  publish (validate contract)        │
        └──────────────┬───────────────────────┘
                       ▼
             [catalog / metadata]
               registry[product] = { schema, owner, slo, piiClass, version }
               search(field="orderId") -> [orders_daily, settlements_daily]
                       │
                       ▼
   consumer query: JOIN orders_daily ON settlements_daily USING(orderId)
   (works only because both contracts expose orderId compatibly)

Global rules (federated): every product MUST have data_owner + piiClass.
Local freedom: storage, transform tooling, internal model per domain.
```

## Stretch Goals

- Add automated contract-diffing in CI that blocks a merge introducing a breaking change.
- Implement a "consumer registration" so producers know who depends on them before they change a schema.
- Add per-product data-quality tests (null rates, referential integrity) that feed the SLO status.

## Definition of Done

- [ ] Each domain publishes an independently owned data product with schema, owner, and SLO.
- [ ] The catalog is searchable and returns ownership and SLO for any product.
- [ ] A backward-incompatible schema change is caught at publish time, not by a broken consumer.
- [ ] A cross-domain join runs using only public contracts.
- [ ] Global governance fields are enforced on every product; a missing one blocks publish.

## Common Pitfalls

- Rebuilding a central warehouse with extra steps — domains that can't publish without a central team aren't autonomous.
- Treating "contract" as documentation instead of an enforced, versioned check.
- No breaking-change policy, so every schema edit silently risks downstream jobs.
- A catalog nobody updates — discoverability decays the moment registration is manual and optional.

## Resources

- [Data Mesh Principles (Zhamak Dehghani)](https://martinfowler.com/articles/data-mesh-principles.html) — the four founding principles.
- [Data Mesh: Logical Architecture](https://martinfowler.com/articles/data-monolith-to-mesh.html) — domains and products explained.
- [Confluent Schema Registry: Compatibility](https://docs.confluent.io/platform/current/schema-registry/fundamentals/avro.html) — how compatibility modes formalize contracts.
- [OpenLineage](https://openlineage.io/docs/) — a standard for describing datasets and their producers/consumers.
