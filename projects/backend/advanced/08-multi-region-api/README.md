# Multi-Region API Design

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

A single-region API is one datacenter fire away from a global outage, and every user on the far side of the planet pays a latency tax on every request. This project has you design and build an API that runs in two or more geographic regions at once: users are routed to the nearest healthy region, data is replicated across regions, and the loss of an entire region triggers an automatic failover the client barely notices. The genuinely hard part is data. The moment the same record can be written in two regions, you inherit replication lag, conflicting writes, and the CAP trade-off in its rawest form. You will pick a replication topology, decide what consistency you can honestly promise, and design conflict resolution — then prove it by killing a region under load.

## Prerequisites

- Solid experience building and deploying stateful HTTP services
- Understanding of the CAP theorem and consistency models (strong, eventual, causal)
- Familiarity with DNS, load balancing, and health checks
- Comfort with database replication concepts (leader/follower, multi-leader)
- Ability to run services in at least two isolated environments (regions, VMs, or containers)

## Learning Objectives

By the end, you should be able to:

- Choose a replication topology (single-leader, multi-leader, leaderless) and justify it
- Route traffic to the nearest healthy region with GeoDNS or anycast and health checks
- Reason honestly about replication lag and what consistency you can promise
- Design and implement conflict resolution for concurrent cross-region writes
- Execute an automatic regional failover and measure the recovery window (RTO/RPO)
- Account for data-residency constraints that pin some data to a region

## Functional Requirements

1. The API must run in at least two regions, each able to serve reads and (per your topology) writes independently.
2. Clients must be routed to the nearest healthy region; an unhealthy region must be removed from rotation automatically.
3. Data written in one region must replicate to the others, and the replication lag must be observable.
4. The system must define its consistency model explicitly (e.g. strong for writes in the leader region, eventual for cross-region reads) and behave accordingly.
5. Concurrent conflicting writes must be resolved by a documented strategy (last-writer-wins with a caveat, version vectors, or CRDTs) — not silently lost.
6. Losing an entire region must trigger failover; in-flight and subsequent requests must succeed against a surviving region.
7. **Availability:** the design must target a documented multi-region availability objective and survive a single-region loss without a full outage.
8. **Consistency vs. latency:** the trade-off must be explicit per endpoint; document which reads may be stale and by how much.
9. **Data residency:** the design must support pinning region-restricted data (e.g. EU users) and never replicate it out of its allowed region.

## Suggested Milestones

1. **Milestone 1 — Two regions, read-local:** Deploy the API in two regions with a shared or replicated read store; route by geography.
2. **Milestone 2 — Replication:** Replicate writes across regions and expose replication lag; pick single- or multi-leader.
3. **Milestone 3 — Conflict resolution:** Force concurrent conflicting writes and resolve them by your chosen strategy.
4. **Milestone 4 — Failover:** Add health checks and automatic failover; kill a region under load and measure RTO/RPO.

## Data & Interface Sketch

```text
Topology

   users (EU) ─┐                                  ┌─ users (US)
               ▼                                   ▼
          [ GeoDNS / anycast + health checks ]
               │                                   │
        ┌──────▼──────┐   async replication ┌──────▼──────┐
        │  Region EU  │◀───────────────────▶│  Region US  │
        │ API + store │   (lag observable)  │ API + store │
        └─────────────┘                     └─────────────┘
             │  residency-pinned data stays in-region only  │

Record (for conflict handling)
  id, value, version/vectorClock, region, updatedAt

Consistency policy (per endpoint)
  writes:  leader-region strong  |  reads: local eventual (staleness bound)
Failover: region health FAIL -> drop from DNS -> traffic shifts -> RTO target
```

## Stretch Goals

- Add read-your-writes consistency for a user by pinning their session to a region.
- Implement CRDT-based state for one data type and compare it to last-writer-wins.
- Add a third region and observe how quorum/replication cost changes.
- Build a chaos test that randomly partitions regions and asserts invariants hold.

## Definition of Done

- [ ] Requests are served from the nearest healthy region and fail over automatically when it dies.
- [ ] A write in one region becomes visible in the others within a measured, documented lag.
- [ ] Concurrent conflicting writes resolve deterministically, with no silent data loss.
- [ ] A killed region causes no full outage; RTO and RPO are measured and reported.
- [ ] Residency-pinned data provably never leaves its allowed region.
- [ ] Each endpoint documents its consistency guarantee and worst-case staleness.

## Common Pitfalls

- Assuming synchronous cross-region replication is free — the speed of light makes it a latency killer; most systems replicate asynchronously.
- "Last-writer-wins" with wall-clock timestamps across regions — clock skew silently drops the wrong write.
- Failing over on a single failed health check, flapping traffic between regions on transient blips.
- Replicating residency-restricted data everywhere for convenience, breaking compliance.
- Promising strong consistency on cross-region reads that are physically eventual, then debugging "impossible" bugs.

## Resources

- [Werner Vogels: Eventually Consistent](https://www.allthingsdistributed.com/2008/12/eventually_consistent.html) — the foundational essay on consistency trade-offs at scale.
- [Martin Kleppmann: Designing Data-Intensive Applications](https://dataintensive.net/) — replication, multi-leader conflicts, and consistency models.
- [Jepsen: Consistency models](https://jepsen.io/consistency) — a precise map of what each consistency level actually guarantees.
- [AWS: Multi-Region fundamentals](https://docs.aws.amazon.com/whitepapers/latest/aws-multi-region-fundamentals/aws-multi-region-fundamentals.html) — patterns for routing, replication, and failover.
- [CRDTs: Conflict-free Replicated Data Types (Shapiro et al.)](https://inria.hal.science/inria-00609399/document) — the paper behind merge-without-conflict data types.
