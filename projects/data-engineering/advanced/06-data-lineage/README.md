# Data Lineage System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build a system that tracks data lineage — which datasets a given table was derived from, through which jobs, and (ideally) at the column level. When a downstream dashboard shows wrong numbers, lineage answers "what feeds this?" in seconds instead of a day of archaeology; when a source column changes, lineage answers "what breaks?" before you ship. You will capture lineage automatically from running jobs (emitting run/dataset/job events, ideally in the OpenLineage format), store it as a directed acyclic graph, and expose queries for upstream/downstream traversal and impact analysis. The core challenges are *completeness* (a job you don't instrument is a blind spot), keeping graph queries fast as the graph grows, and capturing lineage without asking engineers to hand-maintain it.

## Prerequisites

- Familiarity with pipelines/DAGs and an orchestrator (Airflow, Dagster, or similar)
- Comfort with graph modeling and a graph or relational store
- Understanding of jobs producing/consuming datasets
- Basic knowledge of a lineage standard like OpenLineage (helpful, not required)

## Learning Objectives

By the end, you should be able to:

- Model lineage as a DAG of datasets, jobs, and runs
- Capture lineage automatically from job execution rather than manual annotation
- Support upstream ("what feeds X") and downstream ("what depends on X") traversal
- Run impact analysis: given a changed/broken source, list everything affected
- Reason about completeness gaps and how partial lineage misleads

## Functional Requirements

1. Every job run must emit a lineage event recording its input datasets, output datasets, and run status.
2. The system must store lineage as a queryable DAG with nodes for datasets and jobs.
3. A query must return the full upstream chain of any dataset (transitive sources).
4. A query must return the full downstream chain (transitive consumers) for impact analysis.
5. Column-level lineage must be supported for at least one transformation (which input columns produced an output column).
6. The system must record schema/version changes on datasets over time.

## Suggested Milestones

1. **Milestone 1 — Capture:** Emit run/job/dataset events from a couple of pipeline steps and persist them.
2. **Milestone 2 — Graph & traversal:** Build the DAG and expose upstream/downstream queries with cycle protection.
3. **Milestone 3 — Impact & columns:** Add column-level lineage for one transform and an impact-analysis query, plus a lineage visualization.

## Data & Interface Sketch

```text
lineage event (per run):
  { runId, job: "etl.orders_daily", state: complete,
    inputs:  [orders_raw, fx_rates],
    outputs: [orders_daily],
    columnLineage: { "orders_daily.amount_usd":
                       ["orders_raw.amount", "fx_rates.rate"] } }

graph:
  orders_raw ─▶ [etl.orders_daily] ─▶ orders_daily ─▶ [bi.revenue] ─▶ revenue_dash
  fx_rates  ─┘

queries:
  upstream(revenue_dash)   -> {orders_daily, orders_raw, fx_rates}
  downstream(orders_raw)   -> {orders_daily, revenue_dash}   (impact analysis)
  columnsFor(orders_daily.amount_usd) -> [orders_raw.amount, fx_rates.rate]

Non-functional: traversal p95 < T on N nodes; capture adds < X% job overhead;
completeness = instrumented jobs / total jobs (track it).
```

## Stretch Goals

- Integrate OpenLineage so lineage is emitted by real connectors, not just your own hooks.
- Add "freshness propagation": mark all downstream datasets stale when an upstream run fails.
- Detect and alert on orphaned datasets (no producing job) and dead-end sources.

## Definition of Done

- [ ] Job runs emit lineage events automatically; no manual graph editing required.
- [ ] Upstream and downstream traversals return correct transitive sets, cycle-safe.
- [ ] Column-level lineage is correct for at least one non-trivial transformation.
- [ ] Impact analysis lists everything affected by a changed source.
- [ ] A visualization renders the DAG and traversal p95 stays within target as the graph grows.

## Common Pitfalls

- Manual lineage that drifts the moment a job changes — automatic capture is the whole point.
- Silent completeness gaps: an uninstrumented job breaks the chain and nobody notices.
- Storing lineage without run/time versioning, so you can't answer "what did this look like last Tuesday".
- Unbounded traversals with no cycle detection, hanging on a diamond-shaped graph.

## Resources

- [OpenLineage documentation](https://openlineage.io/docs/) — the open standard for lineage events.
- [OpenLineage: Column-level lineage](https://openlineage.io/docs/spec/facets/dataset-facets/column_lineage_facet/) — the facet that models column derivation.
- [Marquez](https://marquezproject.github.io/marquez/) — a reference lineage metadata service built on OpenLineage.
- [Apache Atlas](https://atlas.apache.org/#/) — an enterprise metadata and lineage governance system to compare against.
