# Zero-Downtime Deployment System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Ship new versions of a service while it is serving live traffic, without dropping a single request. You will build a deployment system that shifts traffic gradually to a new version, watches real signals while it does, and rolls back automatically the moment those signals go bad. The core idea is to make a release a controlled experiment rather than a leap of faith: a canary takes a slice of traffic, health and SLO metrics act as promotion gates, and only a healthy canary earns more traffic. You will also confront the parts that quietly cause "zero-downtime" deploys to drop requests anyway — connection draining, readiness gates, and backward-compatible schema changes. The deliverable is a repeatable pipeline where a bad deploy is a non-event.

## Prerequisites

- A containerized service running behind a load balancer or ingress
- Observability that exposes latency and error rate as queryable SLIs
- Familiarity with Kubernetes rollout mechanics or your platform's equivalent
- Understanding of readiness/liveness probes and connection lifecycle

## Learning Objectives

By the end, you should be able to:

- Implement canary and/or blue-green deployments with gradual traffic shifting
- Gate promotion on real SLI metrics, not just "pods are up"
- Roll back automatically when a health or SLO guardrail is breached
- Drain connections and use readiness gates so in-flight requests aren't dropped
- Make schema and API changes backward-compatible across versions

## Functional Requirements

1. A new version must receive traffic gradually, starting from a small percentage.
2. Promotion to more traffic must be gated on measured SLIs (error rate, latency).
3. If a guardrail metric breaches, the system must roll back automatically.
4. No in-flight request may be dropped during a shift or rollback (connection draining).
5. Readiness must gate traffic so a pod receives requests only when truly ready.
6. A rollback must return to the last known-good version without manual manifest surgery.
7. Every deployment must be observable: current split, canary health, and decision.

## Suggested Milestones

1. **Milestone 1 — Gradual shift:** Deploy a new version to a small traffic slice behind the LB.
2. **Milestone 2 — Promotion gates:** Query SLIs and promote only when the canary is healthy.
3. **Milestone 3 — Auto rollback:** Breach a guardrail on purpose and confirm automatic rollback.
4. **Milestone 4 — Safety details:** Add connection draining, readiness gates, and a backward-compatible schema change walkthrough.

## Data & Interface Sketch

```text
   deploy v2
      │
      ▼
   ┌──────────────┐   step 1: 5%   ┌───────────────┐
   │ Rollout       │──────────────▶│ Traffic split │
   │ controller    │   step 2: 25% │  v1 ▓▓▓░  v2 ░ │
   │ (Argo Rollouts│   step 3: 50% └──────┬────────┘
   │  / Flagger)   │   step 4: 100%       │
   └──────┬────────┘                      ▼
          │  query SLIs           ┌───────────────┐
          └──────────────────────▶│ Metrics (SLI) │
          gate: promote if healthy│ err%, latency │
          rollback if breached    └───────────────┘

Release decision per step:
  if error_rate < 1% AND p99 < 300ms for T minutes -> promote
  else -> rollback to v1, drain v2 connections

Non-functional targets:
  request loss       0 dropped requests during shift/rollback
  rollback MTTR      < 60s from breach to full revert
  availability       maintained >= SLO throughout the deploy
```

## Stretch Goals

- Add feature flags so code ships dark and is enabled independently of the deploy.
- Support blue-green in addition to canary and compare their trade-offs on your workload.
- Add automated analysis (multiple metrics, statistical comparison to baseline) for promotion.
- Integrate the rollout into GitOps so the desired version lives in Git and rollout is declarative.

## Definition of Done

- [ ] A new version is promoted gradually with SLI-gated steps.
- [ ] A deliberately bad version is rolled back automatically within the stated MTTR.
- [ ] No requests are dropped during shift or rollback (verified under load).
- [ ] Readiness gates traffic; connection draining is in place.
- [ ] A backward-compatible schema change is demonstrated across two versions.

## Common Pitfalls

- Treating "pods running" as "healthy" and promoting a canary that returns errors fast.
- Skipping connection draining, so rollback drops the very requests you were protecting.
- Breaking schema compatibility, so v1 and v2 can't coexist during the shift.
- No automatic rollback, leaving a human to notice the breach minutes into an incident.
- Canarying by pod count instead of traffic percentage, so the split isn't what you think.

## Resources

- [Argo Rollouts documentation](https://argo-rollouts.readthedocs.io/) — canary and blue-green for Kubernetes.
- [Flagger documentation](https://docs.flagger.app/) — progressive delivery with automated analysis.
- [Martin Fowler: BlueGreenDeployment](https://martinfowler.com/bliki/BlueGreenDeployment.html) — the canonical pattern description.
- [Google SRE Workbook: Canarying Releases](https://sre.google/workbook/canarying-releases/) — safe progressive rollout practice.
