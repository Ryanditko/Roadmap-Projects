# Centralized Logging System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a pipeline that collects logs from several running services, ships them to a central store, and makes them searchable in one place. Instead of SSHing into three machines and grepping files, you should be able to answer "show me every error from the payment service in the last hour" from a single query interface. Pick a stack — ELK/EFK (Elasticsearch + Kibana), or Grafana Loki, or an equivalent — and wire up collection, parsing, storage with a retention policy, and dashboards. The theme is turning scattered, unstructured text into queryable, correlated, bounded data.

## Prerequisites

- Two or more services (or containers) that emit logs, ideally in different formats
- Docker / Docker Compose or a small cluster to host the logging stack
- Understanding of stdout/stderr, log levels, and structured vs plain-text logs
- Basic familiarity with querying and JSON

## Learning Objectives

By the end, you should be able to:

- Ship logs from multiple sources with a collector/agent (Fluent Bit, Fluentd, Filebeat, Promtail)
- Parse and enrich logs into structured fields (level, service, timestamp, trace id)
- Store logs in a searchable backend and query across all sources at once
- Apply a retention policy so storage doesn't grow without bound
- Build a dashboard and at least one alert on a log-derived signal

## Functional Requirements

1. Logs from at least two distinct services must be collected without changing app code beyond structured output.
2. A collector must forward logs to the central store, buffering to avoid loss during backpressure.
3. Logs must be parsed into structured fields, including level, service name, and timestamp.
4. A single query must be able to filter across all services by field (e.g. `level=error AND service=payments`).
5. A retention policy must automatically drop or archive logs older than a defined window.
6. A dashboard must visualize log volume and error rate over time.
7. An alert must fire when the error rate crosses a threshold.

## Suggested Milestones

1. **Milestone 1 — Collect & store:** Get logs from services into the central backend, viewable raw.
2. **Milestone 2 — Parse & query:** Structure logs into fields and query across sources.
3. **Milestone 3 — Retain & observe:** Add retention, a dashboard, and a threshold alert.

## Data & Interface Sketch

```text
Flow:
  service A ─┐
  service B ─┼─> collector/agent ─(buffer)─> store/index ─> query UI + dashboards
  service C ─┘                                    |
                                             retention job (age-out)

Structured log record (target shape):
  timestamp:  ISO-8601
  level:      DEBUG|INFO|WARN|ERROR
  service:    string
  message:    string
  trace_id:   string (optional, for correlation)

Query example (conceptual):
  service = "payments" AND level = "ERROR" AND time > now-1h
```

## Stretch Goals

- Correlate logs across services using a shared trace/request id.
- Add multi-tenancy or role-based access so teams only see their own logs.
- Ship metrics derived from logs (e.g. error count) into a metrics system.
- Add index/stream lifecycle tiers (hot/warm/cold) to control storage cost.

## Definition of Done

- [ ] Logs from all sources appear in the central store within seconds of being emitted.
- [ ] A single query filters by level and service across every source.
- [ ] Logs are structured, not stored as opaque text blobs.
- [ ] Old logs are removed or archived automatically per the retention policy.
- [ ] A dashboard shows error rate and an alert fires when it spikes.

## Common Pitfalls

- Collecting raw text without parsing, so you can grep but never aggregate or alert.
- No buffering in the collector, so a store outage silently drops logs.
- Forgetting retention, letting the index grow until it exhausts disk and takes the store down.
- Inconsistent timestamps/timezones across services, making correlation misleading.
- Logging secrets or PII into a searchable store without redaction.

## Resources

- [Grafana Loki documentation](https://grafana.com/docs/loki/latest/) — log aggregation designed to be cost-efficient.
- [Elastic Stack (ELK) documentation](https://www.elastic.co/guide/index.html) — Elasticsearch, Logstash, Kibana.
- [Fluent Bit documentation](https://docs.fluentbit.io/manual) — lightweight log collection and parsing.
- [Twelve-Factor App: Logs](https://12factor.net/logs) — why apps should treat logs as event streams.
</content>
