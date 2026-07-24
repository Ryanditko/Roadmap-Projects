# Real-time Data Platform

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build an end-to-end real-time data platform: raw events flow in through a durable log, a stream processor turns them into rolling aggregates, and a low-latency serving layer answers dashboard and API queries within seconds of an event landing. Think of the "live metrics" view behind a payments dashboard or a ride-hailing ops screen — the value is entirely in freshness, so a pipeline that is correct but ten minutes stale has failed. This project forces you to reason about the whole path at once: ingestion durability, processing semantics, state, and read-side latency. You will pick where to accept approximation (windowed counts) and where you cannot (money totals), and you will design for the failure modes that only appear when data never stops arriving.

## Prerequisites

- Comfort with a stream processor's core model (Flink, Spark Structured Streaming, or Kafka Streams)
- Experience running a partitioned log like Kafka or Pulsar, including consumer groups and offsets
- A grounding in batch pipelines ([distributed ETL](../02-distributed-etl/) is a useful warm-up)
- Familiarity with windowing, watermarks, and event-time vs processing-time

## Learning Objectives

By the end, you should be able to:

- Design an ingestion → processing → serving topology with explicit delivery guarantees at each hop
- Choose event-time windowing and watermark strategy for out-of-order and late data
- Manage large keyed state and reason about checkpoint/restore cost
- Separate a hot serving store from the processing layer and justify the split
- Define and measure end-to-end latency and freshness SLOs

## Functional Requirements

1. The platform must ingest events into a partitioned, replayable log and survive a broker restart without data loss.
2. A stream job must compute time-windowed aggregations keyed by a business dimension (e.g. per-merchant, per-region).
3. Late events arriving within a bounded allowed-lateness must still update their window; events beyond it must be routed to a side output, not silently dropped.
4. Results must be written to a serving store that answers point and range queries in single-digit milliseconds.
5. The system must expose end-to-end latency (event timestamp → queryable) as a metric.
6. On job restart from checkpoint, aggregates must not double-count already-processed events.

## Suggested Milestones

1. **Milestone 1 — Ingest & replay:** Stand up the log, produce synthetic events with embedded event-time, and prove you can replay from an offset.
2. **Milestone 2 — Windowed processing:** Implement keyed event-time windows with watermarks, checkpointing, and a late-data side output.
3. **Milestone 3 — Serve & observe:** Sink aggregates to the hot store, add a query API, and instrument freshness and latency SLOs.

## Data & Interface Sketch

```text
producers ─▶ [log: topic "events", N partitions, RF=3]
                     │  event: {id, merchantId, amountCents, eventTime}
                     ▼
              [stream job]  keyBy(merchantId)
                 tumbling 1-min windows, watermark = maxEventTime - 30s
                 allowedLateness = 5min ─▶ side output "late"
                 checkpoint every 30s ─▶ durable state backend
                     │  agg: {merchantId, windowStart, count, sumCents}
                     ▼
              [hot store: Redis / key-value]  key = merchantId:windowStart
                     ▼
GET /metrics/{merchantId}?from=..&to=..  -> [{windowStart, count, sumCents}]
GET /health/freshness                    -> { lagSeconds }
```

## Stretch Goals

- Add a second, slower "correction" path (batch reprocessing) and reconcile it against the streaming result — a lambda/kappa comparison.
- Support exactly-once end-to-end by using a transactional sink and idempotent keys.
- Add auto-scaling of processing parallelism driven by consumer lag.

## Definition of Done

- [ ] Events survive a broker or job restart with no loss and no double-counting.
- [ ] Late-but-within-bound events update their window; beyond-bound events land in the side output.
- [ ] The serving API returns windowed aggregates for a key within the target latency.
- [ ] End-to-end freshness lag is exported as a metric and stays under the stated SLO under load.
- [ ] A documented benchmark records throughput, p99 latency, and lag at your target event rate.

## Common Pitfalls

- Mixing processing-time and event-time semantics, so results shift depending on when the job runs.
- Setting watermarks too aggressively and dropping legitimately late data, or too loosely and never closing windows.
- Ignoring state size until checkpoints time out — unbounded keys quietly grow forever.
- Treating the processing store as the serving store, coupling read latency to job restarts.

## Resources

- [Apache Flink: Event Time & Watermarks](https://nightlies.apache.org/flink/flink-docs-stable/docs/concepts/time/) — the canonical model for time in streams.
- [Kafka Documentation: Design](https://kafka.apache.org/documentation/#design) — how the log gives you durability and replay.
- [The Dataflow Model (paper)](https://research.google/pubs/pub43864/) — windowing, watermarks, and triggers, from first principles.
- [Spark Structured Streaming Programming Guide](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html) — an alternative processing model to compare against.
