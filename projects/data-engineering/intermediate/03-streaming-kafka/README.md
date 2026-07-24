# Streaming Pipeline (Kafka)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a real-time streaming pipeline on Apache Kafka: a producer that emits events to a topic, and a consumer that reads them, maintains state, and writes results to a sink. The shift from batch to streaming forces you to confront questions batch never asks — what happens when the consumer crashes mid-message, how do you avoid processing the same event twice, and how do you compute an aggregate over an unbounded stream. You will manage consumer offsets deliberately, design for at-least-once delivery, and make your processing idempotent so duplicates are harmless. You will also add monitoring for consumer lag so you can see when the pipeline falls behind. This project is where "the data is just sitting in a table" stops being true and you start thinking in terms of continuously arriving events.

## Prerequisites

- Comfort with a language that has a Kafka client (Python, Java, or Go)
- Understanding of publish/subscribe messaging and queues
- Familiarity with JSON or another serialization format
- A local Kafka (Docker Compose with Kafka + optionally a schema registry)

## Learning Objectives

By the end, you should be able to:

- Produce and consume events on a partitioned Kafka topic
- Reason about delivery semantics: at-most-once, at-least-once, exactly-once
- Commit offsets deliberately so a crash resumes without losing or replaying data incorrectly
- Make consumer processing idempotent so at-least-once duplicates are harmless
- Compute a windowed aggregate over an unbounded stream and monitor consumer lag

## Functional Requirements

1. A producer must publish structured events to a Kafka topic with a partitioning key.
2. A consumer must read events and write derived results to a sink (database, file, or another topic).
3. The consumer must commit offsets only after a message is successfully processed (at-least-once).
4. Processing must be idempotent so a redelivered message does not corrupt the sink.
5. On restart, the consumer must resume from its last committed offset — no gaps, no full replay.
6. The pipeline must maintain a running aggregate (e.g. per-key count or sum) over a time window.
7. Consumer lag must be observable so falling behind is detectable.

## Suggested Milestones

1. **Milestone 1 — Produce & consume:** Emit events to a topic and log them from a consumer group.
2. **Milestone 2 — Offsets & idempotency:** Commit after processing, then prove a mid-batch crash resumes cleanly.
3. **Milestone 3 — Windowed aggregate & lag:** Maintain a windowed aggregate to the sink and expose consumer lag.

## Data & Interface Sketch

```text
topic: page_views   partitions: 3   key: user_id

event = {
  "event_id":  "uuid",       # dedup key for idempotency
  "user_id":   "u-123",
  "url":       "/pricing",
  "ts":        "2024-01-01T10:00:00Z"
}

Producer --> [ p0 | p1 | p2 ] --> Consumer group "aggregator"
                                     |
                     upsert into sink: (user_id, window) -> count
                     commit offset AFTER upsert succeeds

Delivery: at-least-once + idempotent upsert on event_id  => effectively once
Monitoring: lag = log-end-offset - committed-offset  (per partition)
```

## Stretch Goals

- Add a second consumer instance and watch Kafka rebalance partitions across the group.
- Route un-processable messages to a dead-letter topic instead of blocking the stream.
- Implement tumbling vs sliding windows and compare their output.
- Add a schema registry with Avro and evolve the event schema without breaking consumers.

## Definition of Done

- [ ] Killing the consumer mid-stream and restarting it resumes from the last committed offset.
- [ ] A deliberately redelivered event does not change the aggregate (idempotency verified).
- [ ] The windowed aggregate in the sink matches a hand-computed expected value.
- [ ] Consumer lag is queryable and rises then recovers when the consumer is paused.
- [ ] Two consumers in one group split the partitions without processing the same message twice.

## Common Pitfalls

- Committing offsets before processing, turning a crash into silent data loss (at-most-once by accident).
- Auto-commit on a timer while processing is slow, so offsets advance past unprocessed messages.
- Assuming ordering across partitions — Kafka only orders within a single partition.
- Non-idempotent sink writes, so every rebalance or retry inflates the aggregate.
- Using one partition "for simplicity," then being unable to scale consumers past one.

## Resources

- [Apache Kafka documentation](https://kafka.apache.org/documentation/) — brokers, topics, partitions, and consumer groups.
- [Kafka: Consumer offsets & delivery semantics](https://kafka.apache.org/documentation/#semantics) — at-least-once vs exactly-once.
- [Confluent: Kafka consumers](https://developer.confluent.io/courses/apache-kafka/consumers/) — offset management and rebalancing explained.
- [Confluent: Schema Registry & Avro](https://docs.confluent.io/platform/current/schema-registry/index.html) — evolving event schemas safely.
