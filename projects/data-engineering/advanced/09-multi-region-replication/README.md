# Multi-Region Data Replication

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Design and build a system that replicates data across geographic regions so it survives a whole-region outage and serves reads close to users — while confronting the fact that you cannot have strong consistency, low latency, and partition tolerance all at once. This is where CAP and PACELC stop being trivia and start dictating your design. You will pick a consistency model (synchronous strong replication vs asynchronous eventual, or something in between), define your failure story (RPO — how much data you can lose; RTO — how fast you recover), and handle the messy reality of concurrent writes in two regions producing conflicts. Data residency rules add a constraint that pure engineering can't wave away. The deliverable is a replication design plus a working prototype demonstrating failover and a documented conflict-resolution strategy.

## Prerequisites

- A datastore with cross-region replication features (a distributed DB, Kafka MirrorMaker, or object-store cross-region replication)
- Solid grasp of consistency models (strong, eventual, causal) and the CAP/PACELC theorems
- Understanding of RPO/RTO and failover concepts
- Familiarity with conflict resolution (last-write-wins, vector clocks, CRDTs)

## Learning Objectives

By the end, you should be able to:

- Choose a consistency model and justify it against latency and availability needs
- Define and measure RPO and RTO for a region-loss scenario
- Design and execute a failover (and failback) without data loss beyond your RPO
- Resolve concurrent cross-region write conflicts with a documented strategy
- Account for data-residency constraints in the replication topology

## Functional Requirements

1. Data written in one region must replicate to at least one other region within a bounded lag.
2. The system must define an explicit consistency model and enforce it consistently for reads.
3. A simulated region outage must trigger failover to another region with data loss within the stated RPO.
4. Concurrent writes to the same key in two regions must be reconciled by a documented conflict-resolution rule.
5. Replication lag and per-region health must be observable as metrics.
6. The topology must respect a data-residency rule (e.g. EU-origin data does not leave EU regions).

## Suggested Milestones

1. **Milestone 1 — Replicate:** Stand up two regions and asynchronous replication; measure baseline replication lag.
2. **Milestone 2 — Failover & RPO/RTO:** Simulate a region outage, fail over, measure actual RPO/RTO, then fail back.
3. **Milestone 3 — Conflicts & residency:** Force concurrent conflicting writes, apply a resolution strategy, and enforce a residency constraint.

## Data & Interface Sketch

```text
        region A (primary)                region B (replica/active)
        [write] ──async replicate──▶ replication lag = L
           │                                │
        reads (strong here)             reads (eventual here, unless promoted)

consistency choice (PACELC):
  if Partition -> pick A (availability) or C (consistency)
  Else (normal ops) -> pick L (latency) or C (consistency)

failover: A down -> promote B ; RPO = data written to A not yet replicated
                                RTO = time until B serves writes
conflict (same key, both regions write):
  strategy A: last-write-wins by timestamp  (simple, can lose a write)
  strategy B: version vectors / CRDT         (merges, more complex)

residency: tag data{region_of_origin}; replicate EU-origin only to EU regions.
metrics: replication_lag_ms, region_health, conflict_count.
```

## Stretch Goals

- Implement active-active writes in both regions with CRDT-based merge and prove convergence.
- Add automatic failover with a health-check-driven promotion instead of manual intervention.
- Model quorum-based replication (write to W of N regions) and analyze its RPO/latency profile.

## Definition of Done

- [ ] Writes replicate cross-region within a measured, bounded lag.
- [ ] A region-outage simulation fails over with data loss within the declared RPO and recovery within RTO.
- [ ] Concurrent conflicting writes are reconciled by a documented, tested rule.
- [ ] Replication lag and region health are exported as metrics.
- [ ] A residency constraint is enforced and demonstrated (restricted data never leaves its allowed regions).

## Common Pitfalls

- Claiming "strongly consistent and highly available across regions" — the CAP theorem says pick two under partition.
- Never actually testing failover, so RTO is a guess and the runbook is fiction.
- Last-write-wins with unsynchronized clocks, silently dropping the "losing" write.
- Ignoring residency until an auditor finds EU data replicated to us-east-1.

## Resources

- [Consistency models & CAP (Jepsen)](https://jepsen.io/consistency) — a precise map of consistency guarantees.
- [PACELC theorem](https://en.wikipedia.org/wiki/PACELC_theorem) — the latency-vs-consistency tradeoff even without partitions.
- [Kafka: Geo-replication (MirrorMaker 2)](https://kafka.apache.org/documentation/#georeplication) — cross-cluster/region replication in practice.
- [Amazon: Dynamo (paper)](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) — eventual consistency and conflict resolution at scale.
