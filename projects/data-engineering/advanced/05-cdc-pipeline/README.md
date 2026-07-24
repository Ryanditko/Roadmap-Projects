# CDC Pipeline (Change Data Capture)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build a Change Data Capture pipeline that streams every insert, update, and delete from an operational database into a downstream store — keeping a replica or a data lake continuously in sync without hammering the source with polling queries. The right approach reads the database's write-ahead log (log-based CDC via Debezium), which captures changes in commit order and at low overhead. The genuinely hard parts are ordering (changes to the same row must apply in sequence), the initial snapshot-then-stream handoff (load existing data, then switch to the log without a gap or a duplicate), schema changes mid-stream, and idempotency so a replay after a crash converges to the same state. You will pick delivery semantics — at-least-once with idempotent upserts is usually the pragmatic target — and prove convergence.

## Prerequisites

- A database with logical replication / WAL access (PostgreSQL, MySQL, or MongoDB)
- Familiarity with Kafka or another log for transporting change events
- Understanding of primary keys, upserts, and idempotency
- Comfort with the tradeoffs of exactly-once vs at-least-once delivery

## Learning Objectives

By the end, you should be able to:

- Explain log-based vs query-based vs trigger-based CDC and choose one
- Perform a consistent initial snapshot and hand off to streaming without gaps or duplicates
- Preserve per-key ordering through partitioning
- Handle source schema changes (added/dropped/renamed columns) without breaking consumers
- Design idempotent apply logic so replays converge to identical state

## Functional Requirements

1. The pipeline must capture inserts, updates, and deletes from the source in commit order.
2. An initial snapshot must load existing rows, then transition to streaming with no missed or duplicated changes at the boundary.
3. Changes to the same primary key must be applied in their original order downstream.
4. Applying the change stream must be idempotent: replaying from an earlier offset converges to the same final state.
5. A source schema change must be detected and propagated (or safely rejected), not silently corrupt the target.
6. Replication lag (source commit → applied downstream) must be exposed as a metric.

## Suggested Milestones

1. **Milestone 1 — Capture:** Connect a log-based connector to the source and stream change events into a topic; inspect insert/update/delete envelopes.
2. **Milestone 2 — Snapshot + apply:** Do a consistent snapshot, hand off to streaming, and apply changes as idempotent upserts/deletes into the target.
3. **Milestone 3 — Ordering, schema & lag:** Partition by key for ordering, handle a schema change, and instrument replication lag.

## Data & Interface Sketch

```text
[source DB]  WAL / binlog
     │  log-based capture (Debezium)
     ▼
change event (per row):
  { op: c|u|d, key: {id}, before: {...}, after: {...}, ts_ms, lsn }
     │  produce to topic "cdc.public.orders", partition = hash(key.id)
     ▼
[log]  ordering preserved *within* a partition (same key -> same partition)
     ▼
[sink apply]  op=c/u -> UPSERT by key ; op=d -> DELETE by key   (idempotent)
     target row: { id (pk), ..., _lsn, _updated_at }
     apply rule: ignore event if event.lsn <= stored _lsn  (dedupe on replay)

Snapshot->stream handoff: snapshot at LSN X, then stream from X (no gap).
Delivery: at-least-once transport + idempotent upsert => effectively-once state.
Metric: lag = now - source_commit_ts of last applied event.
```

## Stretch Goals

- Add tombstone handling and compaction so deletes propagate and the topic stays bounded.
- Support a schema-registry-backed contract and evolve a column without downstream breakage.
- Add a reconciliation job that periodically diffs source vs target row counts/checksums.

## Definition of Done

- [ ] Inserts, updates, and deletes all propagate correctly to the target.
- [ ] Snapshot-to-stream handoff produces no gap and no duplicate at the boundary.
- [ ] Same-key changes apply in order; a shuffled-partition test would break, and yours doesn't.
- [ ] Replaying from an old offset converges to identical target state (idempotency proven).
- [ ] Replication lag is exported and stays within the stated bound under a write burst.

## Common Pitfalls

- Query-based CDC (`WHERE updated_at > ?`) that misses deletes and hard-deletes entirely.
- Losing ordering by partitioning on the wrong key, so an update lands before its insert.
- A snapshot that isn't consistent with the log offset, leaving a gap or overlap at handoff.
- Non-idempotent apply, so a crash-replay double-applies and corrupts counts.

## Resources

- [Debezium documentation](https://debezium.io/documentation/reference/stable/index.html) — the reference log-based CDC platform.
- [Debezium: Change event structure](https://debezium.io/documentation/reference/stable/connectors/postgresql.html) — the before/after/op envelope.
- [Kafka Connect](https://kafka.apache.org/documentation/#connect) — how connectors move CDC events into a log.
- [PostgreSQL: Logical Decoding](https://www.postgresql.org/docs/current/logicaldecoding.html) — the WAL mechanism log-based CDC relies on.
