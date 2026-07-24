# Blue/Green Deployment

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Implement a blue/green deployment: run two identical production environments — "blue" (live) and "green" (idle) — deploy the new version to the idle one, validate it in isolation, then flip a router or load balancer to send all traffic to it in a single atomic switch. If anything is wrong, flip back instantly. The value is zero-downtime releases and an instant rollback that doesn't require a rebuild. You will confront the hard parts everyone skips: how to health-check the idle environment before the switch, and what to do about database schema changes that both versions must tolerate.

## Prerequisites

- A deployable app and a router/load balancer you can reprogram (Nginx, HAProxy, cloud LB, or Kubernetes Service)
- Two environments you can run in parallel (containers, VMs, or namespaces)
- Understanding of health checks and how traffic is routed to a backend
- Awareness that the database is usually shared between blue and green

## Learning Objectives

By the end, you should be able to:

- Run two parallel, independently deployable environments behind one entry point
- Deploy and validate a new version without any user traffic reaching it
- Switch all traffic atomically and roll back by switching again
- Health-check the idle environment as a gate before the switch
- Reason about schema changes that must be backward-compatible during the overlap

## Functional Requirements

1. Two environments (blue and green) must be runnable simultaneously behind a single router.
2. A new version must deploy to the idle environment while the live one keeps serving.
3. The idle environment must pass a health/smoke check before it can receive traffic.
4. The traffic switch must be atomic — no request should hit a half-switched state.
5. Rollback must be a single switch back to the previous environment, with no rebuild.
6. Users must observe zero downtime and zero failed requests across the switch.
7. The active environment (blue vs green) must be observable at any time.

## Suggested Milestones

1. **Milestone 1 — Two environments:** Run blue and green behind one router; point it at blue.
2. **Milestone 2 — Deploy & validate idle:** Deploy to green, health-check it while blue serves.
3. **Milestone 3 — Switch & rollback:** Flip traffic to green, verify, then practice an instant rollback.

## Data & Interface Sketch

```text
Topology:
                    ┌──────────┐
   clients ───────> │  router  │ ──(active)──> BLUE  (v1, live)
                    └──────────┘        └─────> GREEN (v2, idle, being validated)

Switch = repoint router upstream from BLUE to GREEN (atomic)
Rollback = repoint back to BLUE

Gate before switch:
  deploy v2 -> GREEN
  run health/smoke checks against GREEN directly (not via router)
  all green? -> switch ; else -> abort, GREEN stays idle

Shared DB caveat:
  schema must satisfy BOTH v1 and v2 during overlap (expand/contract)
```

## Stretch Goals

- Automate the whole flip from a pipeline, with the health gate as a required step.
- Add a canary step: route a small percentage to green before the full switch.
- Model an expand/contract migration so a schema change is safe across the switch.
- Keep the old environment warm for a defined window before reclaiming it.

## Definition of Done

- [ ] Blue and green run side by side with independent versions.
- [ ] A new version is validated on the idle environment before any traffic reaches it.
- [ ] The switch causes zero failed requests (verified with a load generator during the flip).
- [ ] Rollback is a single re-switch with no rebuild and completes in seconds.
- [ ] The currently active environment is always identifiable.

## Common Pitfalls

- Switching before the idle environment is truly ready, so the "zero downtime" release serves errors.
- A non-atomic switch (e.g. editing config on multiple LBs one by one) leaving a split-brain window.
- Forgetting the shared database: a v2-only schema change breaks v1 the instant you'd need to roll back.
- Reclaiming the old environment immediately, destroying your instant-rollback safety net.
- Draining connections abruptly, cutting off in-flight requests at the moment of the switch.

## Resources

- [Martin Fowler: BlueGreenDeployment](https://martinfowler.com/bliki/BlueGreenDeployment.html) — the canonical description of the pattern.
- [AWS: Blue/Green deployments whitepaper](https://docs.aws.amazon.com/whitepapers/latest/blue-green-deployments/welcome.html) — techniques and trade-offs.
- [Expand/contract (parallel change)](https://martinfowler.com/bliki/ParallelChange.html) — safe schema changes across versions.
- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) — one way to model blue/green with Services.
</content>
