# Observability Platform (logs + metrics)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

When a distributed system misbehaves at 3 a.m., the only thing standing between you and a long night is your telemetry. This project has you build the platform that collects it: a pipeline that ingests logs and metrics from many services, stores them efficiently under retention rules, and lets an operator query, chart, and alert on them. The hard part is volume and shape. Logs are high-cardinality text; metrics are compact time series — they demand different storage and query engines. You will design an ingestion path that survives bursts, a time-series store that answers range queries fast, a log index that supports full-text search without exploding on disk, and an alerting engine that fires on thresholds without drowning the on-call in noise. Treat this as building a miniature Prometheus + Loki, not adopting one.

## Prerequisites

- Experience building HTTP/gRPC services and background workers
- Comfort with a message queue or buffer for decoupling ingestion from storage
- Understanding of time-series data and aggregation (rate, percentile, sum-over-time)
- Familiarity with indexing and full-text search concepts (inverted index, tokenization)
- A backend stack of your choice plus any datastore you can shape for time series and text

## Learning Objectives

By the end, you should be able to:

- Design a decoupled ingestion pipeline that absorbs bursts without dropping data
- Store metrics as time series and answer range/aggregation queries efficiently
- Index logs for full-text search while controlling cardinality and disk growth
- Build a rules engine that evaluates alert conditions on a schedule and deduplicates
- Enforce retention and downsampling so storage cost stays bounded
- Correlate logs and metrics through shared labels and a trace/correlation ID

## Functional Requirements

1. The platform must ingest structured logs and numeric metrics from multiple sources via a documented push (or scrape) protocol.
2. Ingestion must be decoupled from storage by a buffer so a storage slowdown does not drop incoming telemetry.
3. Metrics must be stored as labeled time series and queryable by range with aggregation (rate, avg, p95, sum).
4. Logs must be indexed for full-text and label search, with results paginated and time-bounded.
5. Retention policies must expire or downsample old data automatically per configured windows.
6. An alerting engine must evaluate rules on a schedule, fire when a condition holds for a duration, and resolve when it clears.
7. Dashboards must render metric charts and let an operator drill from an alert to the underlying logs.
8. **Scalability:** the design must handle high write throughput and bounded query latency as data volume grows; document how sharding/partitioning by time and label works.
9. **Reliability:** ingestion must degrade gracefully (buffer, backpressure, shed) rather than crash under a telemetry spike.
10. **Observability of itself:** the platform must expose its own ingestion lag, dropped-sample count, and query latency.

## Suggested Milestones

1. **Milestone 1 — Ingestion & buffering:** Accept logs and metrics over an API, push them onto a queue, and persist raw.
2. **Milestone 2 — Metric store & queries:** Store labeled time series and serve range queries with basic aggregations.
3. **Milestone 3 — Log index & search:** Index log lines by labels and full text; serve bounded search queries.
4. **Milestone 4 — Alerting & retention:** Add a scheduled rules engine, alert lifecycle, and retention/downsampling.
5. **Milestone 5 — Dashboards & correlation:** Build charts and drill-down from a firing alert to correlated logs.

## Data & Interface Sketch

```text
Pipeline

  services --push--> [ Ingest API ] --> [ Buffer/Queue ] --> [ Writers ]
                                                              /        \
                                              [ Metric TSDB ]          [ Log Index ]
                                                     |                       |
                                              [ Query API ] <---- [ Dashboards / Alerts ]
                                                     |
                                              [ Rules Engine ] --fires--> [ Notifier ]

Metric point
  name, labels{service, region, ...}, timestamp, value

Log record
  timestamp, level, service, correlationId, message, fields{...}

GET /query/metrics?expr=rate(http_requests_total[5m])&from&to
GET /query/logs?labels={service="api"}&q="timeout"&from&to
POST /rules   { name, expr, forDuration, threshold, notify }
```

## Stretch Goals

- Add distributed-tracing ingestion and visualize a request span tree.
- Implement simple anomaly detection (moving average / z-score) as an alert type.
- Add incident grouping so related alerts collapse into one incident.
- Support downsampling tiers (raw → 1m → 1h) with automatic query routing.

## Definition of Done

- [ ] Logs and metrics from at least two services flow through the buffer into storage.
- [ ] A range query with aggregation returns correct values within a bounded latency.
- [ ] Full-text log search returns matching, time-bounded, paginated results.
- [ ] An alert fires only after its condition holds for the configured duration and later resolves.
- [ ] Old data expires or downsamples per retention policy; storage does not grow unbounded.
- [ ] A telemetry spike is absorbed by the buffer without crashing ingestion.

## Common Pitfalls

- Writing straight to storage on the request path — a slow disk then blocks every producer; buffer first.
- Letting metric label cardinality explode (per-user or per-request labels), melting the time-series store.
- Indexing full log bodies with no retention, so disk grows without bound.
- Alerts that fire on a single spike instead of a sustained condition, training the on-call to ignore them.
- Storing metrics and logs the same way — their access patterns differ, and one engine serves neither well.

## Resources

- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) — the four golden signals and alerting philosophy.
- [Prometheus: Data model and querying basics](https://prometheus.io/docs/concepts/data_model/) — labeled time series and PromQL.
- [Grafana Loki: How it works](https://grafana.com/docs/loki/latest/get-started/overview/) — indexing logs by labels instead of full content.
- [OpenTelemetry documentation](https://opentelemetry.io/docs/) — a vendor-neutral standard for telemetry collection.
- [Gorilla: A Fast, Scalable, In-Memory Time Series Database (Facebook)](https://www.vldb.org/pvldb/vol8/p1816-teller.pdf) — the paper behind modern TSDB compression.
