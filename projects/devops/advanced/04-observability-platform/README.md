# Full Observability Platform

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build the platform that answers "what is happening in production, and why?" by unifying the three pillars — metrics, logs, and traces — behind a correlation story that lets an operator jump from a spiking dashboard to the exact trace to the specific log line. You will instrument services, ship telemetry through a collector, store each signal in an appropriate backend, and tie them together with shared identifiers so a single request can be followed end to end. The advanced challenges are the ones that decide whether this survives contact with real traffic: cardinality control on metrics, sampling on traces, retention and cost at volume, and alerts that page a human only when a human is actually needed. A good observability platform turns debugging from archaeology into a query.

## Prerequisites

- One or more running services you can instrument (ideally with a request path across services)
- Familiarity with Prometheus-style metrics and PromQL basics
- Understanding of structured logging and distributed tracing concepts
- Comfort deploying stateful backends and a telemetry collector

## Learning Objectives

By the end, you should be able to:

- Instrument services for metrics, logs, and traces using open standards (OpenTelemetry)
- Correlate the three signals via trace/span IDs and consistent labels
- Control cardinality, sampling, and retention to keep cost and volume sane
- Build dashboards and alerts driven by SLIs, not vanity metrics
- Design alerting that minimizes false pages while catching real degradation

## Functional Requirements

1. Services must emit metrics, structured logs, and distributed traces via a common pipeline.
2. A single request must be traceable across at least two services with a shared trace ID.
3. A dashboard must present SLIs (latency, error rate, saturation) for each key service.
4. Logs must be searchable and joinable to a trace via a correlation identifier.
5. Alerts must fire on SLO burn, not raw thresholds, and route to a notification channel.
6. The platform must apply sampling and/or cardinality limits to bound cost.
7. Retention policies must be defined per signal and enforced.

## Suggested Milestones

1. **Milestone 1 — Metrics & dashboards:** Scrape metrics, define SLIs, and build a service dashboard.
2. **Milestone 2 — Logs & tracing:** Add structured logs and distributed tracing through a collector.
3. **Milestone 3 — Correlation:** Wire trace IDs into logs so you can pivot signal to signal.
4. **Milestone 4 — Alerting & cost control:** Add SLO-burn alerts, sampling, cardinality limits, and retention policies.

## Data & Interface Sketch

```text
   services (OTel SDK)
      │ metrics    │ logs        │ traces
      ▼            ▼             ▼
   ┌──────────────────────────────────┐
   │   OpenTelemetry Collector        │  (batch, sample, enrich)
   └───────┬──────────┬──────────┬────┘
           ▼          ▼          ▼
     ┌──────────┐ ┌────────┐ ┌────────┐
     │Prometheus│ │  Loki  │ │ Tempo/ │
     │ (metrics)│ │ (logs) │ │ Jaeger │
     └────┬─────┘ └───┬────┘ └───┬────┘
          └──────┬────┴──────────┘
                 ▼      correlate by trace_id + labels
           ┌───────────┐        ┌────────────┐
           │  Grafana  │        │ Alertmanager│──▶ notify
           │ dashboards│        │ (SLO burn) │
           └───────────┘        └────────────┘

Correlation contract:
  every log line carries: trace_id, span_id, service, level
  every metric carries:   service, route, status (bounded label set)

Non-functional targets:
  metric cardinality  bounded (< N series per service)
  trace sampling      head or tail sampling to cap volume
  alert precision     page only on SLO burn; noise ratio tracked
```

## Stretch Goals

- Add exemplars linking a metric spike directly to a representative trace.
- Introduce anomaly detection or multi-window multi-burn-rate SLO alerting.
- Add tenant/team isolation so cost and access are attributable.
- Build a "one-click from alert to root cause" workflow across the three signals.

## Definition of Done

- [ ] All three signals flow through a single collector pipeline.
- [ ] One request is followed end-to-end across services via a trace ID.
- [ ] A log line can be pivoted to its trace and vice versa.
- [ ] Alerts fire on SLO burn and route to a real channel, with false pages minimized.
- [ ] Sampling, cardinality limits, and retention are configured and enforced.

## Common Pitfalls

- Unbounded label cardinality (user IDs, URLs) that explodes the metrics backend and cost.
- Collecting all three signals but never correlating them, so debugging still means three tabs.
- Threshold alerts that page at 3 a.m. for a transient blip nobody needs to act on.
- No sampling on traces, so ingestion and storage cost grows linearly with traffic forever.
- Treating dashboards as the product; the product is fast answers, which need SLIs and correlation.

## Resources

- [OpenTelemetry documentation](https://opentelemetry.io/docs/) — vendor-neutral instrumentation standard.
- [Prometheus documentation](https://prometheus.io/docs/) — metrics collection and PromQL.
- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) — the four golden signals.
- [Google SRE Workbook: Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/) — burn-rate alerting done right.
