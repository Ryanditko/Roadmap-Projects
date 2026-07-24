# Self-Healing Pipelines

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build data pipelines that detect their own failures and recover automatically — retrying transient errors, quarantining poison records, rerouting around a dead dependency, and rolling back a bad run — so a human is paged for genuine novelty, not for the same flaky failure at 3am. The design challenge is doing this *safely*: an over-eager auto-remediation can amplify an incident (retry storms hammering a struggling downstream, a rollback that deletes good data). You will classify failures (transient vs permanent), pick the right response per class (retry with backoff, circuit-break, quarantine, compensate/rollback), and add health checks and anomaly detection to trigger them. The theme is graceful degradation with guardrails. The deliverable is a pipeline that survives injected failures with documented recovery behavior and blast-radius limits.

## Prerequisites

- Experience building pipelines and an orchestrator (Airflow, Dagster, Temporal, or similar)
- Understanding of retry strategies, idempotency, and exponential backoff
- Familiarity with circuit breakers, dead-letter queues, and health checks
- Comfort reasoning about failure modes and blast radius

## Learning Objectives

By the end, you should be able to:

- Classify failures as transient vs permanent and respond appropriately to each
- Implement safe retries with exponential backoff and jitter, bounded by idempotency
- Use circuit breakers and dead-letter/quarantine to contain a failing dependency or bad records
- Design compensation/rollback so a partial failure leaves consistent state
- Detect anomalies (volume, latency, error rate) that trigger remediation before a hard failure

## Functional Requirements

1. The pipeline must distinguish transient failures (retryable) from permanent ones (not) and act differently on each.
2. Transient failures must retry with exponential backoff and jitter, capped, and only where operations are idempotent.
3. Records that repeatedly fail must be quarantined to a dead-letter store, not block the whole pipeline.
4. A failing downstream dependency must trip a circuit breaker that stops hammering it and recovers when it heals.
5. A failed run must roll back or compensate so no partial, inconsistent output is published.
6. Health checks and at least one anomaly signal must be able to trigger automated remediation, with every action logged.

## Suggested Milestones

1. **Milestone 1 — Retry & classify:** Add failure classification and idempotent retry-with-backoff; inject transient errors and watch recovery.
2. **Milestone 2 — Contain:** Add a dead-letter/quarantine path and a circuit breaker for a flaky dependency.
3. **Milestone 3 — Rollback & detect:** Add compensation/rollback for a bad run and an anomaly detector that triggers remediation, all audited.

## Data & Interface Sketch

```text
                 ┌─────────── failure classifier ───────────┐
   [stage] ──err─▶ transient? ──yes──▶ retry(backoff, jitter, maxN)  (idempotent only)
                 └ permanent? ──yes──▶ quarantine record ─▶ [dead-letter store]
                                        (pipeline keeps flowing)

dependency call ─▶ [circuit breaker]  closed -> allow | open -> fail fast + skip
                     trips at error_rate > threshold; half-open probes to recover

run outcome:
  success -> atomic publish
  failure -> compensate/rollback -> no partial output visible

anomaly detector: watch {input_volume, latency, error_rate}
   deviation > k*stddev -> trigger remediation + PAGE if unrecognized

guardrails: max retries, breaker cooldown, remediation rate-limit
audit log: every automated action { time, trigger, action, result }
```

## Stretch Goals

- Add a "learning" layer that tracks recurring failure signatures and suggests (or applies) a known fix.
- Implement automatic backfill of a quarantined batch once the root cause clears.
- Add chaos testing that randomly injects failures in CI to prove the healing actually works.

## Definition of Done

- [ ] Transient and permanent failures are classified and handled differently.
- [ ] Idempotent retries with capped backoff+jitter recover from injected transient errors.
- [ ] Poison records land in a dead-letter store without stalling the pipeline.
- [ ] A circuit breaker protects a flaky dependency and recovers automatically.
- [ ] A failed run leaves no partial output; every automated remediation is logged and rate-limited.

## Common Pitfalls

- Retry storms: unbounded or un-jittered retries turn a blip into a self-inflicted outage.
- Retrying non-idempotent operations and double-applying side effects.
- Auto-healing so aggressively it masks a real bug — nobody ever learns the pipeline is broken.
- Rollback that deletes or corrupts good data because compensation wasn't scoped to the failed run.

## Resources

- [Google SRE Book: Handling Overload & Cascading Failures](https://sre.google/sre-book/handling-overload/) — retry budgets and load shedding done right.
- [AWS: Timeouts, retries, and backoff with jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/) — the canonical safe-retry guidance.
- [Martin Fowler: Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html) — the pattern for containing a failing dependency.
- [Airflow: Retries & callbacks](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/tasks.html) — orchestrator-level retry and failure hooks.
