# Incremental Data Processing

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a batch job that processes only the records that are new or changed since its last successful run, instead of reprocessing the entire source every time. This is the workhorse pattern behind nightly warehouse loads and CDC pipelines: you track a high-water mark (a timestamp or monotonic id), read the delta above it, and advance the mark only after the write commits. The interesting part is not the SQL — it is the bookkeeping. What happens when the job crashes mid-write, when a row arrives late with an old timestamp, or when you need to rerun yesterday? Getting those cases right is the difference between a pipeline you trust and one you babysit.

## Prerequisites

- Comfort writing batch transformations over a tabular source (a warehouse table, files, or an API)
- Understanding of timestamps, watermarks, and monotonic identifiers
- Familiarity with idempotency — running the same operation twice yields the same result
- An engine of your choice (SQL + a scheduler, Spark, DuckDB, or a plain script)

## Learning Objectives

By the end, you should be able to:

- Detect new and changed rows using a watermark instead of a full scan
- Persist and advance processing state safely across runs
- Make writes idempotent so a retry never double-counts
- Distinguish full-refresh from incremental load and choose between them
- Handle late-arriving data and support backfilling a past window

## Functional Requirements

1. The job must read only records with a change key greater than the last committed watermark.
2. The job must persist the new watermark only after the corresponding write succeeds.
3. Rerunning the job with no new source data must produce no changes (idempotent).
4. The job must support a full-refresh mode that rebuilds the target from scratch.
5. The job must accept an explicit date/id range to backfill or reprocess a past window.
6. Upserts must key on the record's natural id so a changed row updates in place, not duplicates.
7. The job must record run metadata: rows read, rows written, watermark before/after, and status.

## Suggested Milestones

1. **Milestone 1 — Full load:** Read the whole source and write the target once, capturing the initial watermark.
2. **Milestone 2 — Delta load:** Read only rows above the watermark, upsert them, and advance the mark after commit.
3. **Milestone 3 — Recovery & backfill:** Make reruns idempotent, add a backfill range, and handle late arrivals with a lookback window.

## Data & Interface Sketch

```text
source.orders
  id           bigint    (natural key)
  updated_at   timestamp (change key / watermark source)
  ...payload

pipeline_state
  pipeline     string    (e.g. "orders_incremental")
  watermark    timestamp
  updated_at   timestamp

run flow:
  1. read W  = state.watermark
  2. rows    = source where updated_at > W - lookback   (lookback catches late data)
  3. upsert rows into target on id
  4. W' = max(updated_at) among rows
  5. commit target, then set state.watermark = W'

modes: incremental (default) | full-refresh | backfill(from, to)
```

## Stretch Goals

- Add hard-delete handling by consuming a change stream or comparing against a snapshot.
- Track per-run lineage so you can answer "which run produced this row?".
- Detect and alert when the watermark stops advancing (a stuck pipeline).
- Support parallel workers by partitioning the delta range and committing state atomically.

## Definition of Done

- [ ] A second run with no new data writes nothing and leaves the watermark unchanged.
- [ ] Killing the job after the write but before the state commit, then rerunning, produces no duplicates.
- [ ] A changed source row updates the existing target row rather than inserting a new one.
- [ ] Backfilling a past range reprocesses exactly that window without disturbing newer data.
- [ ] Run metadata is persisted and readable for every execution.

## Common Pitfalls

- Advancing the watermark before the write commits — a crash then silently skips rows forever.
- Using `>=` on the watermark and reprocessing the boundary row, or `>` and dropping it — pick one and stay consistent.
- Assuming `updated_at` is strictly increasing; clock skew and late writes mean you need a lookback buffer.
- Appending instead of upserting, so changed rows accumulate as duplicates.
- Making full-refresh truncate the target before the new load succeeds, leaving an empty table on failure.

## Resources

- [Airbyte: Incremental sync modes](https://docs.airbyte.com/using-airbyte/core-concepts/sync-modes/incremental-append) — how a real tool models incremental vs full loads.
- [dbt: Incremental models](https://docs.getdbt.com/docs/build/incremental-models) — watermark-based incremental builds in practice.
- [Martin Kleppmann: Designing Data-Intensive Applications](https://dataintensive.net/) — chapters on change capture and derived data.
- [Wikipedia: Change data capture](https://en.wikipedia.org/wiki/Change_data_capture) — the broader family of techniques.
