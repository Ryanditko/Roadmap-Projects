# Design a Multi-Region Architecture

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Advanced · **Estimated time:** 3–7 days

## Overview

Design a system that runs across multiple geographic regions so that users everywhere get low latency, and the loss of an entire region does not take the product down. The hard part is not spinning up servers in two continents — it is the data layer. The moment you replicate state across regions, you inherit the CAP trade-off: replication takes tens to hundreds of milliseconds, so you must decide, per data type, whether you tolerate stale reads, resolve write conflicts, or pay the latency for synchronous consistency. This is a design exercise — the deliverable is a design document with diagrams, not deployed infrastructure.

## Prerequisites

- Understanding of the CAP theorem and consistency models (strong, eventual, causal)
- Familiarity with database replication (leader-follower, multi-leader, quorum)
- Comfort with DNS/global load balancing and health-based routing
- Basic capacity and cost estimation across regions
- An intermediate distributed-systems project as a stepping stone (e.g. [Design a CDN](../../intermediate/10-cdn/))

## Learning Objectives

By the end, you should be able to:

- Estimate cross-region traffic, replication bandwidth, and the latency floor set by geography
- Choose a consistency model per data type and justify it against the latency cost
- Design global request routing with regional failover and health checks
- Design a disaster-recovery plan with explicit RTO and RPO targets
- Reason about data residency/compliance constraints that pin certain data to a region

## Requirements & Constraints

**Functional**

- Serve reads and writes from the region nearest the user where possible.
- Replicate durable state across regions and resolve conflicting concurrent writes.
- Route users to a healthy region and fail over automatically when one becomes unavailable.
- Support region-pinned data for residency/compliance (e.g. EU user data stays in the EU).

**Non-functional**

- Availability target 99.99%; survive the loss of one full region with no data loss for critical data.
- Define RTO (time to recover) and RPO (acceptable data loss) per data class.
- Read latency p99 within regional budget; state the cross-region write penalty explicitly.
- Cost-aware: cross-region replication bandwidth and duplicated capacity are the main cost drivers.

## Suggested Approach

1. **Estimate first.** Pick a scenario (e.g. 3 regions, 100k writes/sec globally, 1 KB average record). Derive replication bandwidth, cross-region RPS, and the geographic latency floor (speed-of-light RTT).
2. **Classify data by consistency need.** Split into strong (e.g. account balances), eventual (e.g. user profiles, feeds), and region-pinned (compliance). This classification drives every later decision.
3. **Choose replication topology per class.** Single global leader vs. multi-leader vs. quorum; sketch conflict resolution (last-writer-wins, CRDTs, or app-level merge).
4. **Design global routing.** GeoDNS or Anycast plus health checks; define the failover trigger and how in-flight writes are handled.
5. **Design DR.** Define RTO/RPO per data class, the failover runbook, and how you test it (game days).

## Architecture Sketch

```text
                    Global routing (GeoDNS / Anycast + health checks)
                       │                                    │
          ┌────────────▼───────────┐          ┌─────────────▼──────────┐
          │   Region A (us-east)    │          │   Region B (eu-west)   │
          │  App tier               │          │  App tier              │
          │  Regional cache         │          │  Regional cache        │
          │  DB replica ◄───────────┼──async/sync replication──────────┤
          └─────────────────────────┘          └────────────────────────┘
                       │  (region-pinned writes stay local for compliance)
                    Region C (ap-south) ... same shape

Replicated record
  key:        string
  value:      bytes
  version:    vector clock | Lamport ts   (for conflict detection)
  region:     home region id
  residency:  GLOBAL | EU_ONLY | ...       (compliance tag)
  updatedAt:  epoch ms

Control decisions
  route(user)          -> nearest healthy region
  write(key, val)      -> local leader; replicate per consistency class
  onRegionDown(region) -> reroute traffic; promote replica if leader lost
  conflict(a, b)       -> resolve via LWW | CRDT merge | app rule
```

## Deep-Dive Topics

- **Consistency models:** Strong vs. eventual vs. causal; where each fits and the latency it costs.
- **Conflict resolution:** Last-writer-wins vs. CRDTs vs. application merge — correctness vs. complexity.
- **Global routing & failover:** DNS TTL effects on failover speed; Anycast draining; split-brain avoidance.
- **RTO/RPO and DR testing:** Setting targets per data class and validating them with game days.
- **Data residency:** Pinning data to a region and the routing implications for a traveling user.

## Deliverables

- A design document (~4–6 pages) covering routing, replication, consistency, and DR.
- Capacity estimation: cross-region replication bandwidth, RPS, and the geographic latency floor, with assumptions.
- A data-classification table mapping each data type to its consistency model and residency rule.
- The architecture diagram, replicated data model, and control-plane decision contract.
- A DR section with explicit RTO/RPO targets per data class and a failover runbook outline.

## Common Pitfalls

- Applying one consistency model to all data — either paying cross-region latency everywhere or corrupting critical state.
- Ignoring the speed-of-light latency floor; no amount of engineering makes synchronous cross-continent writes fast.
- Multi-leader replication without a conflict-resolution strategy, producing silent data divergence.
- Setting RTO/RPO on paper but never running a failover drill, so the runbook is wrong when it matters.
- Forgetting data-residency law; replicating EU user data to another region can be a compliance violation.

## Resources

- [System Design Primer: CAP theorem](https://github.com/donnemartin/system-design-primer#cap-theorem) — the core consistency/availability trade-off.
- [Jepsen: Consistency models](https://jepsen.io/consistency) — a precise map of strong-to-eventual consistency.
- [AWS: Disaster recovery strategies](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html) — RTO/RPO and DR patterns.
- [Amazon Aurora Global Database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html) — a concrete cross-region replication reference.
- [CRDTs (conflict-free replicated data types)](https://crdt.tech/) — conflict resolution for multi-leader writes.
