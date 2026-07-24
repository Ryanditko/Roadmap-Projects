# Design a Metrics & Monitoring System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Design a system that collects numeric metrics from services — request rates, latencies, CPU — stores them as time series, and lets you query and alert on them. Think a simplified Prometheus plus Grafana. Unlike logs, metrics are small, regular, and numeric, which invites aggregation: you rarely need every raw point forever, so you downsample old data. The dominant risk is cardinality explosion, where too many label combinations blow up storage. You'll reason about the collection model, time-series storage, and rollups. Deliver a design document covering ingestion, storage, and querying.

## Prerequisites

- Understanding of what a metric is (a number sampled over time)
- Awareness of metric types: counters, gauges, histograms
- Familiarity with the idea of a time series (a value stream keyed by labels)
- Comfort reasoning about aggregation over time windows

## Learning Objectives

By the end, you should be able to:

- Design a metric collection model (push vs. pull) and justify it
- Reason about time-series storage and why it differs from a generic database
- Understand cardinality and how labels can explode storage
- Design downsampling and retention to bound cost
- State a trade-off between resolution and storage cost

## Requirements & Constraints

1. Collect metrics from many services at a regular interval.
2. Store them as time series identified by a name plus labels.
3. Query by name, label filters, and time range, with aggregation (avg, p99, rate).
4. Downsample old data to keep storage bounded.
5. Support alerting when a metric crosses a threshold.
6. Estimate scale: 1M active time series, scraped every 15s — roughly 67K samples/s.

## Suggested Approach

1. Choose push vs. pull collection and note the trade-off (pull gives the server control; push suits short-lived jobs).
2. Do the math: 1M series ÷ 15s ≈ 67K samples/s; each sample is tiny, but the series count drives memory and index size.
3. Design the series identity: metric name + sorted label set = one series; reason about how labels multiply cardinality.
4. Design storage that appends samples per series and downsamples older windows.
5. Sketch the query and alerting path: evaluate a rule on recent data on a schedule.

## Architecture Sketch

```text
Services ── expose /metrics ──> [ Scraper ] ──> [ Time-Series Store ]
     (pull, every 15s)                               │  downsample old windows
                                                [ Rollup / Long-term Store ]
Engineer ── query ──> [ Query Engine ] ──> series      [ Alerter ] --rule--> notify

Core API
  GET  /metrics                (exposition, per service)
  GET  /query   ?expr=&from=&to=&step=   -> time series
  POST /alerts  { expr, threshold, for } -> alert rule

Data model
  series id = metric_name + {label1=v1,...}   (labels drive cardinality)
  sample: series_id | ts | value
  retention: raw 15s for 7d -> 5m rollup for 90d -> 1h rollup for 1y
```

## Deep-Dive Topics

- **Cardinality explosion:** how a high-cardinality label (like user ID) multiplies series into millions and wrecks storage.
- **Push vs. pull:** service discovery and health signal with pull, vs. push for ephemeral batch jobs.
- **Downsampling:** aggregating raw samples into coarser rollups so a year of history stays affordable.

## Deliverables

- An architecture diagram showing scraping, time-series storage, query, and alerting.
- The query and alert-rule API contract.
- A data model for series identity, samples, and the retention/rollup scheme.
- One trade-off written up: e.g., high-resolution long retention (rich history, expensive storage) vs. aggressive downsampling (cheap, but old data loses fine detail).

## Common Pitfalls

- Adding an unbounded label (user ID, request ID) and detonating cardinality.
- Storing metrics in a generic relational DB and hitting write-throughput and query limits.
- Keeping full-resolution data forever instead of downsampling, so cost grows without bound.
- Confusing metrics with logs — metrics are aggregatable numbers, not searchable text.

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — storage and scaling patterns.
- [Prometheus: Overview](https://prometheus.io/docs/introduction/overview/) — the reference pull-based metrics system.
- [Prometheus: Data model](https://prometheus.io/docs/concepts/data_model/) — series, labels, and cardinality.
- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) — what to measure and alert on.
