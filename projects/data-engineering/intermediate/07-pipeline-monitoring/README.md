# Pipeline Monitoring System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build the observability layer that sits alongside your data pipelines and answers the question every data engineer dreads at 9am: "did last night's load actually work?" You will capture each pipeline run as a first-class record — start time, end time, status, row counts — evaluate it against SLA and freshness expectations, and fire an alert when something is late, empty, or broken. The goal is to catch a silent failure before a stakeholder does. The hard part is tuning: too sensitive and everyone mutes the channel, too lax and you find out about a broken table from a dashboard three days later.

## Prerequisites

- A pipeline or two whose runs you can instrument (even simple scripts on a schedule)
- Understanding of SLAs, data freshness, and the idea of an alerting threshold
- Familiarity with time-series thinking (a metric observed over successive runs)
- A store for run history (a table or document collection) and any notification channel

## Learning Objectives

By the end, you should be able to:

- Model a pipeline run as a structured, queryable event
- Define and evaluate SLA and freshness rules against run history
- Detect anomalies such as a sudden drop in row count or an overrun in duration
- Route alerts with enough context to act, and suppress duplicate noise
- Present pipeline health at a glance for someone who is not on call

## Functional Requirements

1. Every pipeline run must be recorded with start time, end time, status, and rows processed.
2. The system must flag a run that fails, exceeds its expected duration, or produces zero rows.
3. The system must evaluate data freshness — alert when a dataset has not updated within its SLA window.
4. The system must detect a row-count anomaly relative to a rolling baseline (e.g. drop > 50%).
5. An alert must include the pipeline name, what rule tripped, the observed value, and the expected value.
6. Repeated alerts for the same ongoing failure must be deduplicated, not resent every run.
7. The system must expose a health view listing each pipeline's last run, status, and freshness.

## Suggested Milestones

1. **Milestone 1 — Capture runs:** Instrument pipelines to emit a run record and store the history.
2. **Milestone 2 — Rules & alerts:** Add SLA, freshness, and row-count rules that produce alerts with context.
3. **Milestone 3 — Noise control & dashboard:** Deduplicate alerts and build a health view over the run history.

## Data & Interface Sketch

```text
pipeline_run
  run_id        uuid
  pipeline      string
  started_at    timestamp
  ended_at      timestamp
  status        enum(success | failed | running)
  rows_out      integer

rule types:
  sla_duration   -> ended_at - started_at > threshold
  freshness      -> now - last_success.ended_at > window
  volume_anomaly -> rows_out < baseline * (1 - drop_pct)

alert
  pipeline, rule, observed, expected, first_seen, still_open
  -> notify to <channel>; dedupe while still_open

health view: pipeline | last_run | status | age | open_alerts
```

## Stretch Goals

- Add root-cause hints by correlating a failure with the upstream pipeline that feeds it.
- Track a per-run trace so a slow stage inside a pipeline is visible, not just total duration.
- Support alert severities and escalation (warn in chat, page on repeated critical).
- Auto-resolve an alert and post a recovery notice when the next run succeeds.

## Definition of Done

- [ ] Every run appears in the history with correct status and row count.
- [ ] A failed or empty run produces exactly one alert, not one per retry.
- [ ] A dataset that goes stale past its SLA triggers a freshness alert without a run failing.
- [ ] A sudden row-count drop is flagged against the rolling baseline.
- [ ] The health view reflects the true last-run state for each pipeline.

## Common Pitfalls

- Alerting on every run so the channel becomes noise everyone ignores — deduplicate open incidents.
- Measuring freshness from run start instead of successful completion, hiding partial failures.
- Using a fixed row-count threshold that breaks on seasonal traffic; prefer a rolling baseline.
- Sending alerts with no context, forcing the reader to go dig for what actually broke.
- Treating a still-running long job as a failure because you only check for a completed record.

## Resources

- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) — the four golden signals and alerting philosophy.
- [Prometheus: Alerting best practices](https://prometheus.io/docs/practices/alerting/) — how to write rules that stay actionable.
- [Airflow: SLAs and monitoring](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/tasks.html#slas) — SLA concepts in a real orchestrator.
- [Monte Carlo: What is data observability](https://www.montecarlodata.com/blog-what-is-data-observability/) — freshness, volume, and schema as observability pillars.
