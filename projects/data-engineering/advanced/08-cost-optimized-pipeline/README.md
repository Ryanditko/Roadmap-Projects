# Cost-Optimized Data Pipeline

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Take a working but wasteful data pipeline and cut its cloud bill without breaking its SLAs. This is an optimization project where the objective function is dollars, and the discipline is measuring cost per run *before* you touch anything. You will attribute spend across compute, storage, and data transfer, then attack the biggest line item: spot/preemptible instances for fault-tolerant batch work, storage tiering and compression, avoiding cross-region egress, right-sizing over-provisioned clusters, and scheduling non-urgent jobs into cheaper windows. The constant tension is cost vs performance and cost vs reliability — a spot instance is cheap until it's reclaimed mid-job. The deliverable is a before/after cost analysis with each optimization's savings and its risk trade-off documented.

## Prerequisites

- A cloud data stack you can measure (managed Spark, a warehouse, or object storage + query engine)
- Understanding of cloud pricing dimensions: compute-hours, storage class, egress, requests
- Familiarity with spot/preemptible instances and their reclamation behavior
- Comfort reading a cost/billing breakdown and attributing it to workloads

## Learning Objectives

By the end, you should be able to:

- Instrument and attribute pipeline cost across compute, storage, and transfer
- Use spot/preemptible capacity for fault-tolerant work without risking data loss
- Apply storage tiering, compression, and file layout to cut storage and scan costs
- Eliminate avoidable data-transfer/egress charges
- Weigh each optimization's savings against its performance and reliability risk

## Functional Requirements

1. The pipeline's cost per run must be measured and attributed to compute, storage, and transfer before optimization.
2. At least one fault-tolerant stage must run on spot/preemptible capacity with safe handling of reclamation (checkpoint/retry).
3. Storage cost must be reduced via tiering and/or compression, with data still queryable within SLA.
4. A documented change must eliminate or reduce cross-region/egress transfer cost.
5. Every optimization must preserve the pipeline's correctness and its latency/freshness SLA.
6. A budget/cost alert must fire when spend exceeds a defined threshold.

## Suggested Milestones

1. **Milestone 1 — Measure & attribute:** Instrument cost per run and break it down by resource; identify the top 2–3 line items.
2. **Milestone 2 — Optimize compute & storage:** Move a stage to spot with checkpointing; apply tiering/compression; re-measure.
3. **Milestone 3 — Transfer & guardrails:** Cut egress, add scheduling into cheap windows, and wire a budget alert.

## Data & Interface Sketch

```text
cost attribution (per run, before):
  compute  $X  (cluster hours x instance price)   <- usually the big one
  storage  $Y  (GB-month x storage class)
  transfer $Z  (cross-region egress GB x rate)
  total    $T  --> target: reduce to $T' at same SLA

optimizations & risk:
  on-demand -> spot (batch)   save ~60-90%   risk: reclamation -> need checkpoint+retry
  standard  -> tiered storage save on cold    risk: retrieval latency on archive
  cross-region -> same-region  save egress    risk: reduced geo-redundancy
  right-size cluster           save idle       risk: less burst headroom
  schedule off-peak            save (spot mkt) risk: later completion

guardrail: if month_to_date_spend > BUDGET -> alert (do not silently overrun)
invariant: correctness + SLA unchanged after every change.
```

## Stretch Goals

- Add a cost-vs-latency Pareto chart so a stakeholder can pick a point, not just "cheapest".
- Implement automatic right-sizing that scales the cluster to observed load per run.
- Model committed-use / reserved discounts and compute the break-even utilization.

## Definition of Done

- [ ] Cost per run is measured and attributed before and after, with total savings quantified.
- [ ] A spot-backed stage survives reclamation via checkpoint/retry with no data loss.
- [ ] Storage and/or scan cost drops measurably while data stays queryable within SLA.
- [ ] An egress/transfer optimization is documented with its savings.
- [ ] A budget alert fires on threshold breach; each optimization's risk is written down.

## Common Pitfalls

- Optimizing cost you can't see — no attribution means you cut the wrong thing.
- Putting a stateful, non-recoverable stage on spot and losing work when it's reclaimed.
- Compressing/tiering so aggressively that a query blows its latency SLA fetching cold data.
- Ignoring egress until the transfer line dominates the bill — cross-region reads are silent money.

## Resources

- [AWS Well-Architected: Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html) — a structured framework for cost decisions.
- [AWS: Spot Instances best practices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-best-practices.html) — using interruptible capacity safely.
- [Google Cloud: Storage classes](https://cloud.google.com/storage/docs/storage-classes) — tiering economics and retrieval tradeoffs.
- [Spark: Tuning](https://spark.apache.org/docs/latest/tuning.html) — right-sizing resources to avoid paying for idle.
