# Design a Centralized Logging System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Design a system that collects logs from many services, stores them centrally, and lets engineers search them — a simplified ELK stack or Loki. The defining trait is write-heavy, append-only, high-volume data: logs pour in constantly, are rarely updated, and are searched occasionally. That shape drives every decision, from how logs are shipped and buffered to how they're indexed and aged out. You'll reason about ingestion throughput, indexing cost, and retention. Deliver a design document covering the collection pipeline, storage, and search.

## Prerequisites

- Understanding of what a log line is and why services emit them
- Awareness that log volume can be enormous and bursty
- Familiarity with the idea of an index that makes search fast
- Comfort reasoning about the cost of storing data over time

## Learning Objectives

By the end, you should be able to:

- Design a collection pipeline that buffers bursts without losing logs
- Reason about the trade-off between indexing everything and storage cost
- Design a storage layout partitioned by time
- Plan retention and archival to control cost
- State a trade-off between search speed and ingestion/storage cost

## Requirements & Constraints

1. Collect logs from many services and centralize them.
2. Absorb bursts without dropping logs (buffering/backpressure).
3. Support search by time range, service, and level, plus free-text.
4. Apply retention: hot logs searchable, old logs archived or deleted.
5. Ingestion must not slow down the services producing logs.
6. Estimate scale: 500K log lines/s peak, ~500 bytes each — roughly 20 TB/day.

## Suggested Approach

1. Split the pipeline: agents ship logs, a buffer absorbs bursts, an indexer writes to storage.
2. Do the math: 500K lines/s × 500 bytes ≈ 250 MB/s ≈ 20 TB/day; retention of 7 days ≈ 140 TB hot.
3. Design the buffer (a queue or log broker) so a slow indexer never blocks producers.
4. Choose a time-partitioned storage layout so old data is cheap to drop.
5. Decide what gets indexed — full text is powerful but expensive; structured fields are cheaper.

## Architecture Sketch

```text
Services ──agent──> [ Buffer / Broker ] ──> [ Indexer ] ──> [ Hot Store (indexed) ]
                       (absorbs bursts)                            │  age out
                                                          [ Cold / Archive Store ]
Engineer ── query ──> [ Search API ] ──> Hot Store

Core API
  POST /ingest   (batch of log lines)          -> 202
  GET  /search   ?q=&service=&level=&from=&to= -> [ matching lines ]

Data model
  log line: ts | service | level | message | trace_id | fields{}
  storage partitioned by (day, service); index on ts, service, level
```

## Deep-Dive Topics

- **Backpressure:** how a buffer/broker decouples bursty producers from a steadier indexer, and what happens when the buffer fills.
- **Indexing cost:** full-text inverted index vs. indexing only structured fields; the storage and write amplification of each.
- **Retention tiers:** hot (fast search) vs. cold/archive (cheap, slow), and lifecycle policies to move data between them.

## Deliverables

- An architecture diagram showing agents, buffer, indexer, and hot/cold storage.
- The ingestion and search API contract.
- A data model for a log line and the storage partitioning scheme.
- One trade-off written up: e.g., index every field (fast, flexible search, high storage and write cost) vs. index only key fields (cheap, but some queries require a slow scan).

## Common Pitfalls

- Writing logs synchronously into the store from the producing service, coupling app latency to logging.
- Indexing everything as full text and blowing past the storage budget.
- No retention policy, so hot storage grows without bound and costs explode.
- No buffer, so a burst either drops logs or backs up into the services.

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — data ingestion and storage patterns.
- [Elastic: The ELK Stack](https://www.elastic.co/elastic-stack) — a reference logging architecture.
- [Grafana Loki: Architecture](https://grafana.com/docs/loki/latest/get-started/architecture/) — a cost-focused logging design that indexes labels, not full text.
- [Twelve-Factor App: Logs](https://12factor.net/logs) — treating logs as event streams.
