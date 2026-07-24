# Chaos Engineering Setup

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Deliberately break your own system — under control — to learn how it fails before your users teach you the hard way. You will build a chaos engineering practice: form a hypothesis ("if a pod dies, the service stays within its latency SLO"), inject a real fault, measure the impact against a steady-state baseline, and either confirm resilience or file the weakness you found. The discipline that separates chaos engineering from "randomly killing things" is the scientific method plus blast-radius control: every experiment must have a defined scope, an automatic abort condition, and a way to stop instantly. The goal is not chaos for its own sake; it is confidence, backed by evidence, that your system degrades gracefully.

## Prerequisites

- A running application with meaningful dependencies (a datastore, a downstream service) to disrupt
- Observability in place — metrics and ideally traces — so you can measure steady state
- Comfort with Kubernetes or your target platform's failure primitives
- Understanding of SLIs/SLOs so you know what "still healthy" means

## Learning Objectives

By the end, you should be able to:

- Define steady-state hypotheses in measurable SLI terms
- Inject faults (pod kills, latency, network partition, resource exhaustion) safely
- Enforce blast-radius limits and automatic abort conditions
- Measure experiment impact and distinguish signal from noise
- Turn discovered weaknesses into tracked, fixed resilience improvements

## Functional Requirements

1. Each experiment must declare a steady-state hypothesis in terms of a measurable SLI.
2. The system must inject at least three distinct fault types (e.g. pod kill, latency, partition).
3. Every experiment must have a bounded blast radius (namespace, percentage, or label selector).
4. An automatic abort must halt the experiment if a guardrail metric breaches a threshold.
5. There must be a single, reliable "stop everything now" control.
6. Results must be recorded: hypothesis, fault, observed impact, pass/fail, follow-up.
7. Experiments must be repeatable so a fix can be verified against the same fault.

## Suggested Milestones

1. **Milestone 1 — Baseline & hypothesis:** Establish steady-state SLIs and write your first hypothesis.
2. **Milestone 2 — First injection:** Kill a pod within a tight blast radius, measure recovery, record the result.
3. **Milestone 3 — Guardrails:** Add automatic abort on SLO breach and a global halt switch.
4. **Milestone 4 — Fault library & schedule:** Add latency, partition, and resource faults; run experiments on a cadence and track findings to closure.

## Data & Interface Sketch

```text
   ┌──────────────┐   1. read baseline    ┌───────────────┐
   │  Experiment  │──────────────────────▶│ Observability │
   │  definition  │                       │ (SLIs/metrics)│
   └──────┬───────┘                       └──────┬────────┘
          │ 2. inject fault                      │ 4. compare
          ▼   (bounded blast radius)             │    vs steady state
   ┌──────────────┐                              ▼
   │ Chaos engine │      guardrail breach?  ┌───────────┐
   │ (Chaos Mesh/ │◀────── abort/halt ──────│  Verdict  │
   │  LitmusChaos)│                         │ pass/fail │
   └──────────────┘                         └───────────┘

Experiment record:
  hypothesis:  "p99 latency stays < 300ms if 1 of 3 pods dies"
  fault:       pod-kill, selector app=api, 33% of replicas
  blast_radius: namespace=staging, max 1 pod
  abort_if:    error_rate > 5% OR p99 > 600ms
  result:      pass | fail + follow-up ticket

Non-functional targets:
  blast radius   never exceeds declared scope
  abort latency  < 10s from breach to halt
  MTTR observed  recorded for every failure mode tested
```

## Stretch Goals

- Progress from staging to a controlled game day in production with a small blast radius.
- Add automated, scheduled experiments in CI so regressions in resilience are caught.
- Inject application-level faults (dependency errors, slow responses) not just infra faults.
- Correlate experiment results with real past incidents to prioritize which faults to test.

## Definition of Done

- [ ] At least three fault types run with a declared, enforced blast radius.
- [ ] Every experiment has an automatic abort and a working global halt.
- [ ] Steady-state hypotheses are measured against real SLIs, not guesses.
- [ ] Discovered weaknesses are tracked and at least one has been fixed and re-verified.
- [ ] Experiment results are recorded in a repeatable, reviewable format.

## Common Pitfalls

- Running chaos with no steady-state baseline, so you can't tell if the fault mattered.
- No abort condition — a "small" experiment cascades into a real outage.
- Testing only pod kills, missing the latency and partition faults that cause real incidents.
- Treating a passed experiment as permanent; systems change, so experiments must recur.
- Running in prod before the guardrails and culture are ready, eroding trust in the practice.

## Resources

- [Principles of Chaos Engineering](https://principlesofchaos.org/) — the foundational definition and method.
- [Chaos Mesh documentation](https://chaos-mesh.org/docs/) — a CNCF chaos platform for Kubernetes.
- [LitmusChaos documentation](https://docs.litmuschaos.io/) — open-source chaos engineering framework.
- [Google SRE Book: Embracing Risk](https://sre.google/sre-book/embracing-risk/) — error budgets and reasoning about failure.
