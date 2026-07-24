# Multi-Region Deployment

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Serve users from more than one geographic region so that a regional outage does not become a global one, and so that a user in São Paulo isn't paying a transatlantic round-trip on every request. You will deploy the same application to at least two regions, route users to the nearest healthy region, and decide — deliberately — how state is handled across them. The genuinely hard decisions live in the data layer: do you run active-active with a globally replicated store and accept eventual consistency, or active-passive with a clear failover and an accepted recovery point? You will also confront data residency: some data legally cannot leave its region. This project is about making those trade-offs explicit and provable, not about copy-pasting a stack into two clouds.

## Prerequisites

- Comfort deploying an application to a single region (containers or VMs)
- Understanding of DNS, TLS, and how global load balancing works
- Familiarity with database replication concepts and consistency models
- Awareness of latency, RPO, and RTO as measurable quantities

## Learning Objectives

By the end, you should be able to:

- Deploy an identical application footprint across multiple regions
- Route users to the nearest healthy region and fail over when one degrades
- Choose and justify an active-active vs. active-passive data strategy
- Reason about consistency, RPO, and RTO for cross-region state
- Handle data residency constraints in routing and storage

## Functional Requirements

1. The application must run in at least two regions serving the same functionality.
2. Users must be routed to the nearest healthy region (latency- or geo-based).
3. If a region becomes unhealthy, traffic must fail over to another region automatically.
4. The data strategy (replication or partition) must be explicit with a stated consistency model.
5. The system must define and measure RPO and RTO for a regional failure.
6. Region-restricted data must never be served from or stored in a disallowed region.
7. Failback to a recovered region must be controlled, not an instant full cutover.

## Suggested Milestones

1. **Milestone 1 — Two regions live:** Deploy identical stacks in two regions behind a global entry point.
2. **Milestone 2 — Geo routing & health:** Route by proximity and health-check each region.
3. **Milestone 3 — Data strategy:** Implement replication or partitioning; document consistency, RPO, RTO.
4. **Milestone 4 — Failover drill & residency:** Simulate a regional outage, measure recovery, and enforce residency rules.

## Data & Interface Sketch

```text
                    ┌──────────────────────────┐
       users  ─────▶│  Global DNS / Anycast LB  │  (latency/geo routing + health)
                    └───────┬───────────┬──────┘
                            ▼           ▼
                   ┌───────────┐   ┌───────────┐
                   │ Region A  │   │ Region B  │
                   │ (us-east) │   │ (sa-east) │
                   │  app+cache│   │  app+cache│
                   └─────┬─────┘   └─────┬─────┘
                         │  data layer   │
              active-active │            │ active-passive
              (replicated,  ◀────────────▶ (primary/replica,
               eventual)      replication   failover promote)

Residency rule (example):
  data tagged region=BR  -> only stored/served from sa-east

Non-functional targets:
  availability   >= 99.99% globally (survives 1 region loss)
  RTO            < 5 min to shift traffic + promote data
  RPO            <= replication lag (state and defend it)
  cross-region latency budget stated per request path
```

## Stretch Goals

- Add a third region and test quorum-based writes or read-local/write-global patterns.
- Automate failback with a canary re-entry to avoid a cold region taking full load.
- Add per-region cost tracking and route cost-aware when SLOs allow.
- Run a scheduled regional failover game day and publish the measured RTO/RPO.

## Definition of Done

- [ ] The app serves traffic from two or more regions with proximity routing.
- [ ] A simulated regional outage fails over within the stated RTO.
- [ ] The data strategy is documented with an explicit consistency model, RPO, and RTO.
- [ ] Residency-restricted data is provably never served from a disallowed region.
- [ ] Failback is controlled and observable.

## Common Pitfalls

- Deploying to two regions but pointing both at one region's database — no real isolation.
- Ignoring replication lag, then losing data on failover because RPO was never measured.
- Assuming DNS failover is instant; caches and TTLs make it anything but.
- Treating data residency as an afterthought and discovering a compliance violation in prod.
- Never actually testing failover, so the "multi-region" system fails when a region does.

## Resources

- [AWS Well-Architected: Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html) — multi-region reliability patterns.
- [Google SRE Book: Managing Critical State](https://sre.google/sre-book/managing-critical-state/) — consistency and replication trade-offs.
- [Jepsen: consistency models](https://jepsen.io/consistency) — a precise map of consistency guarantees.
- [Cloudflare: what is Anycast?](https://www.cloudflare.com/learning/cdn/glossary/anycast-network/) — how global routing directs users.
