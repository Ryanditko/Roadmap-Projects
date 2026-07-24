# Auto-Scaling Setup

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build an auto-scaler that adds and removes capacity in response to load, the way a cloud Auto Scaling Group or the Kubernetes Horizontal Pod Autoscaler does. You will scrape a metric (CPU, memory, or a custom signal like queue depth or requests-per-second), compare it against a target, and decide how many instances the system should run right now. The hard parts are not the arithmetic but the control theory around it: cooldown windows so you don't thrash, minimum and maximum bounds so a bad metric can't scale you to zero or bankrupt you, and honest handling of cold starts, where new capacity isn't useful the instant it appears. Done well, the system rides load smoothly; done naively, it oscillates and pages you at 3am.

## Prerequisites

- Comfort with a metrics source (Prometheus, CloudWatch, or a scraped endpoint)
- A workload you can scale — replicas of a container, VMs, or worker processes
- Understanding of what "utilization" means for your chosen metric
- Familiarity with a control loop pattern (observe → decide → act)
- A stepping stone: [Service Restart Monitor](../../beginner/10-service-restart/) covers the health-check loop this builds on

## Learning Objectives

By the end, you should be able to:

- Select and normalize a scaling metric and reason about why it reflects real load
- Implement target-tracking: compute desired replicas from current utilization vs a target
- Apply separate scale-up and scale-down behavior with cooldown to prevent flapping
- Enforce min/max bounds and safety guards against bad or missing metrics
- Account for cold starts so freshly added capacity is not counted as productive too early

## Functional Requirements

1. The scaler must read a metric on a fixed interval from a real source.
2. It must compute a desired instance count using target tracking (`desired = current * metric / target`).
3. Scale-up and scale-down must respect independent cooldown periods.
4. Instance count must never leave the configured `[min, max]` range.
5. Missing, stale, or clearly invalid metrics must not trigger scaling; the last known good state holds.
6. New instances must pass a readiness check before counting toward capacity.
7. Every scaling decision (metric, desired, actual, reason) must be logged for later inspection.

## Suggested Milestones

1. **Milestone 1 — Metric loop:** Scrape one metric on an interval and log current utilization.
2. **Milestone 2 — Target tracking:** Compute desired replicas and actuate scale-up/down within bounds.
3. **Milestone 3 — Stability:** Add cooldowns, readiness gating, and guards against bad metrics.

## Data & Interface Sketch

```text
policy
  metric        cpu | memory | custom(name)
  target        number        (e.g. 60 for 60% CPU)
  min, max      int
  cooldown      { up_s, down_s }
  step_max      int           (max change per decision)

control loop (every interval)
  1. read metric M for current N instances
  2. desired = ceil(N * (M / target))
  3. clamp desired to [min, max] and to +/- step_max
  4. if scaling up and last-up within up_s   -> hold
     if scaling down and last-down within down_s -> hold
  5. actuate; new instances excluded until ready

decision log entry
  ts, metric_value, N, desired, applied, reason
```

## Stretch Goals

- Add predictive scaling from a rolling trend so capacity leads load instead of chasing it.
- Support multiple metrics and scale on the most demanding one.
- Add scheduled scaling for known daily peaks alongside reactive scaling.
- Emit a cost estimate per decision so scale-up is visibly tied to spend.

## Definition of Done

- [ ] A sustained load increase scales the system up within one cooldown window and stops at max.
- [ ] Load dropping scales it back down, never below min.
- [ ] Rapid metric swings do not cause flapping — cooldown demonstrably suppresses oscillation.
- [ ] A missing or absurd metric reading holds state instead of scaling wildly.
- [ ] New instances only count once ready, and every decision is logged with its reason.

## Common Pitfalls

- Scaling on a lagging metric (e.g. average CPU) that reacts too slowly, so you always scale late.
- Symmetric cooldowns: scaling down as eagerly as up causes thrash under bursty traffic.
- Ignoring cold starts, so the scaler adds more capacity while the last batch is still warming up.
- No maximum bound, letting a runaway metric or a metrics outage scale you into a huge bill.
- Trusting a single scrape; one bad sample triggers a scale event that the next sample reverses.

## Resources

- [Kubernetes: Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) — the target-tracking algorithm in detail.
- [AWS: Target Tracking Scaling Policies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scaling-target-tracking.html) — a production auto-scaler's model.
- [Prometheus: Querying Basics](https://prometheus.io/docs/prometheus/latest/querying/basics/) — how to pull a metric to scale on.
- [Google SRE Book: Handling Overload](https://sre.google/sre-book/handling-overload/) — why bounds and graceful behavior matter under load.
