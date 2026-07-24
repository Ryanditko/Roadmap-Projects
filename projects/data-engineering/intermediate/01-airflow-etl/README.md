# ETL Pipeline with Airflow

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a scheduled ETL pipeline that extracts from a source (an API or an operational database), transforms the data, and loads it into a warehouse or analytics table — all orchestrated by Apache Airflow. The interesting part is not the transform itself but the orchestration around it: expressing dependencies as a DAG, scheduling daily runs, and making each run *idempotent* so a re-run for a given date produces the same result instead of duplicating rows. You will add a sensor that waits for upstream data to land before the DAG proceeds, wire up retries and alerting so a transient failure heals itself, and support a backfill that reprocesses a range of historical dates. By the end you will understand why data teams reach for a scheduler instead of a cron job and a pile of shell scripts.

## Prerequisites

- Comfort writing Python and reasoning about functions with side effects
- Basic SQL and an understanding of what a batch job is
- Familiarity with cron-style scheduling concepts
- A local Airflow environment (Docker Compose or `airflow standalone`) — do not install into a shared cluster

## Learning Objectives

By the end, you should be able to:

- Model a data workflow as a DAG with explicit task dependencies
- Parameterize tasks by the logical execution date so runs are isolated and reproducible
- Make a load idempotent so re-running a date never duplicates data
- Use a sensor to wait for an upstream dependency before proceeding
- Backfill a historical date range and reason about task retries and alerting

## Functional Requirements

1. The pipeline must be defined as an Airflow DAG with a daily schedule and clear task dependencies.
2. Each task must derive its input and output partition from the run's logical (execution) date, not `now()`.
3. A load for a given date must be idempotent — re-running it replaces that date's rows rather than appending duplicates.
4. A sensor or equivalent must block the DAG until the required source data for that date is available.
5. Failed tasks must retry with backoff a bounded number of times, and exhausted retries must trigger an alert.
6. The DAG must support a backfill over an arbitrary date range without manual editing per date.
7. The pipeline must record run metadata (rows processed, duration) for later inspection.

## Suggested Milestones

1. **Milestone 1 — Linear DAG:** Extract → transform → load as three ordered tasks running on a daily schedule.
2. **Milestone 2 — Idempotent load & sensor:** Make the load overwrite-by-date, and gate it behind a readiness sensor.
3. **Milestone 3 — Reliability & backfill:** Add retries with backoff, failure alerting, run metrics, then backfill a week.

## Data & Interface Sketch

```text
DAG: daily_sales_etl   schedule=@daily   start_date=2024-01-01

  wait_for_source (sensor: is source/{{ ds }} present?)
        |
     extract  --> raw/date={{ ds }}/data.parquet
        |
    transform --> staging table (partition = ds)
        |
      load    --> DELETE WHERE dt = {{ ds }}; INSERT ...   (idempotent)
        |
  record_metrics --> runs(dag_id, ds, rows, seconds)

{{ ds }} = Airflow logical execution date (YYYY-MM-DD), NOT "today"
Task config: retries=3, retry_delay=5m, sla=2h, on_failure_callback=notify
Backfill: airflow dags backfill -s 2024-01-01 -e 2024-01-07 daily_sales_etl
```

## Stretch Goals

- Add a branching task that skips the load when the extract yields zero rows.
- Generate the DAG dynamically from a config file listing multiple source tables.
- Use dynamic task mapping to fan out over partitions in parallel.
- Add an SLA so a run that misses its deadline raises a distinct alert.

## Definition of Done

- [ ] The DAG renders in the Airflow UI with correct dependencies and no import errors.
- [ ] Re-running any single date produces identical output — verified by row counts before and after.
- [ ] The load never starts until the readiness sensor confirms source data exists.
- [ ] A backfill over several days completes and populates one correct partition per date.
- [ ] A forced task failure retries per config and fires the configured alert.

## Common Pitfalls

- Using `datetime.now()` inside tasks instead of the execution date, which breaks backfills and reproducibility.
- Appending on load so re-runs silently double the data — always make the write idempotent.
- Doing heavy computation in the DAG file's top-level scope, which runs on every scheduler parse.
- Setting `retries` without `retry_delay`, so a flapping dependency hammers the source.
- Treating retries as a substitute for idempotency; a retry of a non-idempotent task corrupts data.

## Resources

- [Apache Airflow: Core Concepts](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/index.html) — DAGs, tasks, operators, and the scheduler.
- [Airflow: Best Practices](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html) — idempotency, top-level code, and testing.
- [Astronomer: Airflow DAG best practices](https://www.astronomer.io/docs/learn/dag-best-practices) — practical patterns for reliable pipelines.
- [The Data Engineer roadmap](https://roadmap.sh/data-engineer) — where orchestration fits in the wider picture.
