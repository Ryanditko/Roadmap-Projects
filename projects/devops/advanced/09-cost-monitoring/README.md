# Cost Monitoring System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Turn an opaque cloud bill into an actionable, attributable, forward-looking view of spend. You will ingest cost and usage data, allocate it to teams and services via tagging, surface waste and anomalies, forecast where the bill is heading, and put guardrails in place so a runaway resource is caught before it shows up as a five-figure surprise. The advanced substance is in the allocation and the incentives: cost data is messy, tags are inconsistent, shared resources resist clean chargeback, and "just turn it off" ignores the reliability the spend buys. A good cost system does not just report numbers — it attributes them to owners, explains the trend, and makes the cost of a decision visible at the moment the decision is made.

## Prerequisites

- Access to a cloud provider's cost and usage export (or a realistic sample dataset)
- Understanding of your resource inventory: compute, storage, network, managed services
- Familiarity with tagging/labeling and how it drives allocation
- Basic data modeling and dashboarding skills

## Learning Objectives

By the end, you should be able to:

- Ingest and normalize cloud cost and usage data
- Allocate spend to teams/services via a tagging strategy, including shared costs
- Detect waste (idle, oversized, orphaned resources) and cost anomalies
- Forecast spend and compare against budgets
- Set guardrails and alerts that catch runaway cost early without blocking legitimate work

## Functional Requirements

1. The system must ingest cost and usage data on a schedule and store it queryably.
2. Spend must be allocatable to a team or service, with a defined rule for shared/untagged costs.
3. The system must flag waste: idle, oversized, or orphaned resources.
4. Cost anomalies (sudden spikes vs. baseline) must be detected and alerted.
5. The system must forecast spend for the current period and compare to a budget.
6. A budget breach or forecast-to-breach must trigger an alert to the owner.
7. Reports must be attributable — every significant cost has an owner or a documented shared bucket.

## Suggested Milestones

1. **Milestone 1 — Ingest & model:** Load cost/usage data and model it for querying.
2. **Milestone 2 — Allocation:** Apply a tagging strategy and attribute spend to owners, handling shared costs.
3. **Milestone 3 — Waste & anomalies:** Detect idle/oversized resources and spike anomalies.
4. **Milestone 4 — Forecast & guardrails:** Forecast the period, compare to budgets, and alert on breach risk.

## Data & Interface Sketch

```text
   cloud billing export (daily)
      │
      ▼
   ┌───────────────┐   normalize + tag-based allocation
   │ Cost ingester  │──────────────┐
   └───────────────┘               ▼
                            ┌───────────────┐
                            │ Cost store    │  (service, team, resource, $)
                            └──────┬────────┘
        ┌──────────────┬──────────┼───────────┬──────────────┐
        ▼              ▼          ▼           ▼              ▼
   allocation     waste       anomaly     forecast       budget
   by team/svc    finder      detector    (period $)     guardrail
        └──────────────┴──────────┴───────────┴──────────────┘
                            ▼
                     ┌───────────┐   alert owner (name@example.com)
                     │ dashboard  │   on anomaly / budget breach
                     └───────────┘

Allocation record:
  service: checkout   team: payments   cost: $/day
  shared: load-balancer -> split by request share (documented rule)

Non-functional targets:
  attribution coverage  >= 95% of spend mapped to an owner
  anomaly detection     spike flagged within 24h
  forecast accuracy     tracked (forecast vs. actual delta)
```

## Stretch Goals

- Add a chargeback/showback report per team with trends and top drivers.
- Recommend rightsizing and commitment (reserved/savings-plan) opportunities with payback estimates.
- Add unit-economics metrics (cost per request, per tenant) so scaling decisions are cost-aware.
- Support multi-cloud so allocation and anomalies span providers.

## Definition of Done

- [ ] Cost/usage data is ingested on a schedule and queryable.
- [ ] At least 95% of spend is attributed to an owner, with a documented rule for shared cost.
- [ ] Waste and cost anomalies are detected and alerted.
- [ ] Spend is forecast for the period and compared against a budget.
- [ ] A budget-breach risk triggers an alert to the owner.

## Common Pitfalls

- Chasing raw cost totals with no allocation, so no one owns or acts on the number.
- Inconsistent tagging that leaves a large "unallocated" bucket nobody investigates.
- Anomaly alerts with no baseline, firing on normal end-of-month or scaling patterns.
- Forecasting off a short window, so seasonality makes every forecast wrong.
- Optimizing cost in isolation and quietly degrading reliability the spend was buying.

## Resources

- [FinOps Framework](https://www.finops.org/framework/) — the discipline of cloud financial management.
- [AWS Well-Architected: Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html) — allocation and rightsizing patterns.
- [Google Cloud: Cost management overview](https://cloud.google.com/cost-management) — budgets, alerts, and reporting concepts.
- [Kubecost / OpenCost](https://www.opencost.io/) — open-source Kubernetes cost allocation.
