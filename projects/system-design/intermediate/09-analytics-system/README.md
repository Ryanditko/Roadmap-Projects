# Design an Analytics System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Design the backend of a product analytics system like Mixpanel or Google Analytics: client apps emit a firehose of behavioral events, the system ingests them reliably, and analysts query aggregates ("daily active users", "funnel conversion") over billions of rows without waiting minutes. The design splits into a high-throughput ingestion path and a query-optimized analytical store, usually with both a real-time and a batch layer. This is a design exercise: you produce a design document, capacity numbers, and diagrams — not working code.

## Prerequisites

- Understanding of event streams and OLTP vs. OLAP workloads
- Familiarity with columnar storage and pre-aggregation at a conceptual level
- Awareness of message queues and stream processing
- Comfort estimating event volume, ingestion throughput, and storage

## Learning Objectives

By the end, you should be able to:

- Design an ingestion pipeline that buffers and batches a high-volume event stream
- Estimate event write QPS, raw storage, and columnar-store storage over time
- Choose a columnar/analytical store and design pre-aggregations for common queries
- Design a lambda-style split between real-time and batch layers
- Justify trade-offs between raw-event storage and pre-aggregated rollups

## Requirements & Constraints

- Assume 1M events/s at peak, avg 500 bytes each, 90-day raw retention, dashboards refreshed near-real-time.
- Ingestion must not lose events if the analytical store is briefly unavailable.
- Common dashboard queries must return in under ~1 s over billions of events.
- Support both predefined dashboards and slower ad-hoc queries.
- Estimate ingestion throughput (MB/s), raw storage, and rollup storage.

## Suggested Approach

1. Compute event ingest bandwidth (events/s × size) and 90-day raw storage.
2. Design ingestion: collector → durable queue → stream processor → columnar store.
3. Design pre-aggregation: roll up events into hourly/daily cubes for dashboards.
4. Split real-time (approximate, recent) from batch (exact, historical) layers.
5. Partition the analytical store by time and a high-cardinality dimension.

## Architecture Sketch

```text
Clients -> [Collector API] -> Kafka (durable buffer, partitioned by eventType)
                                 |-> Stream processor -> real-time rollups (last hours)
                                 |-> Batch loader     -> Columnar store (ClickHouse/BigQuery-like)
                                                            |-> daily rollup tables (cubes)

Dashboard query -> [Query svc] -> rollup tables (fast) OR raw events (ad-hoc, slower)

POST /collect   { userId, event, props{}, ts } -> 202 (accepted, async)
GET  /metrics?metric=DAU&from&to&groupBy       -> 200 { series[] }

Event  { eventId, userId, type, props{}, ts }   // partition by (day, type)
Rollup { day, dimension, metric -> value }      // pre-aggregated cube
```

## Deep-Dive Topics

- **Ingestion durability:** buffering in a queue so a downstream outage never drops events.
- **Pre-aggregation:** rollup cubes vs. querying raw events; cardinality explosion of group-bys.
- **Trade-off 1 — raw events vs. pre-aggregated rollups:** keeping raw events allows any future query but is expensive to store and slow to scan; rollups are tiny and fast but only answer predefined questions. Justify keeping raw for 90 days plus rollups for dashboards.
- **Trade-off 2 — real-time vs. batch accuracy:** the streaming layer gives fresh but approximate numbers; the batch layer is exact but delayed. Justify serving recent windows from streaming and historical from batch.

## Deliverables

- [ ] A design document (~3–5 pages) expanding the pipeline and architecture above.
- [ ] Capacity estimates: ingest MB/s, raw storage (90 days), rollup storage.
- [ ] A partitioning plan for the event and rollup stores (by time + dimension).
- [ ] A caching/pre-aggregation strategy for dashboard queries.
- [ ] At least two trade-offs, each with the option chosen and why.

## Common Pitfalls

- Writing events straight to the analytical store, so any downstream hiccup loses data.
- Storing only rollups, then being unable to answer a new question about past behavior.
- Ignoring group-by cardinality, so a rollup on user-agent explodes to millions of rows.
- Using a row store for analytics, making full scans hopelessly slow.

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — pipelines and storage trade-offs.
- [Lambda Architecture (Marz)](http://nathanmarz.com/blog/how-to-beat-the-cap-theorem.html) — the batch + speed layer pattern.
- [ClickHouse: why columnar](https://clickhouse.com/docs/en/intro) — analytical columnar storage explained.
- [Martin Kleppmann, *Designing Data-Intensive Applications*](https://dataintensive.net/) — batch and stream processing.
