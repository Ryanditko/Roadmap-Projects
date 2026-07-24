# Monitoring Stack (Prometheus + Grafana)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Stand up a metrics-based observability stack: instrument an application to expose metrics, have Prometheus scrape and store them as time series, visualize them in Grafana, and route alerts through Alertmanager when something crosses a threshold. This is the "know your system is healthy before your users tell you" project. You will move from raw counters to meaningful signals — request rate, error rate, and latency (the RED method) — and learn why alerting on symptoms beats alerting on causes.

## Prerequisites

- An application you can instrument (or that already exposes Prometheus metrics)
- Docker / Docker Compose or a cluster to run Prometheus, Grafana, and Alertmanager
- Understanding of counters, gauges, and histograms as metric types
- Basic familiarity with querying and dashboards

## Learning Objectives

By the end, you should be able to:

- Instrument an app with counters, gauges, and histograms exposed on a `/metrics` endpoint
- Configure Prometheus to discover and scrape targets
- Write PromQL queries for rate, error ratio, and latency percentiles
- Build a Grafana dashboard from those queries
- Define alerting rules and route notifications through Alertmanager

## Functional Requirements

1. The target app must expose metrics in the Prometheus exposition format on an HTTP endpoint.
2. Prometheus must scrape the target on an interval and store the series.
3. The app must expose at least request rate, error count, and request-duration histogram.
4. A Grafana dashboard must show the RED signals (Rate, Errors, Duration) over time.
5. At least one alerting rule must be defined (e.g. error ratio above a threshold for N minutes).
6. Alertmanager must deliver a firing alert to a notification channel.
7. Recording or alerting rules must use `rate()`/`histogram_quantile()` correctly over a time window.

## Suggested Milestones

1. **Milestone 1 — Instrument & scrape:** Expose `/metrics`, get Prometheus scraping it.
2. **Milestone 2 — Visualize:** Write PromQL for RED signals and build a Grafana dashboard.
3. **Milestone 3 — Alert:** Add an alert rule and route it through Alertmanager to a channel.

## Data & Interface Sketch

```text
Flow:
  app /metrics ──scrape──> Prometheus (TSDB) ──query(PromQL)──> Grafana dashboards
                                 |
                            eval alert rules ─> Alertmanager ─> notify (email/Slack/webhook)

Metric types on /metrics:
  http_requests_total{method,status}      counter
  http_request_duration_seconds           histogram (buckets)
  process_resident_memory_bytes           gauge

RED queries (conceptual PromQL):
  rate:     sum(rate(http_requests_total[5m]))
  errors:   sum(rate(http_requests_total{status=~"5.."}[5m]))
  duration: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

## Stretch Goals

- Add exporters (node_exporter, cAdvisor) for host/container metrics alongside app metrics.
- Add recording rules to precompute expensive queries used by dashboards.
- Configure alert routing with severities, grouping, and silences in Alertmanager.
- Provision dashboards and datasources as code so the stack is reproducible.

## Definition of Done

- [ ] Prometheus shows the target as "up" and stores its series.
- [ ] A Grafana dashboard displays rate, error ratio, and p95 latency.
- [ ] PromQL uses `rate()` over a range, not raw counter values.
- [ ] An alert transitions to firing when the condition holds and reaches a channel.
- [ ] Restarting the app does not produce false alerts from counter resets.

## Common Pitfalls

- Graphing a raw counter instead of its `rate()`, producing an ever-rising, meaningless line.
- Alerting on every transient blip with no `for:` duration, creating alert fatigue.
- High-cardinality labels (user id, request id) blowing up Prometheus memory.
- Misreading histogram percentiles by applying `histogram_quantile` without `rate()` on the buckets.
- Alerting on causes (CPU high) rather than symptoms (users seeing errors), so pages don't map to impact.

## Resources

- [Prometheus documentation](https://prometheus.io/docs/introduction/overview/) — data model, scraping, and PromQL.
- [Grafana documentation](https://grafana.com/docs/grafana/latest/) — dashboards and data sources.
- [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) — routing, grouping, and silencing alerts.
- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) — signals worth alerting on.
</content>
