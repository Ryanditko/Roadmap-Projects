# Partitioned Data Pipeline

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a pipeline that writes its output split into partitions — typically by date — so that reads scan only the slices they need and reprocessing touches one partition instead of the whole table. This is the layout under every large data lake: `dt=2026-07-24/` folders, partition pruning at query time, and per-partition retention. You will choose a partition key, write data into the right partition, make each partition independently rebuildable, and prove that queries prune away the partitions they do not need. The payoff is concrete: a backfill or a bad-day fix becomes a one-partition operation instead of a full reload.

## Prerequisites

- Comfort writing a batch job that produces a dataset (files or warehouse tables)
- Understanding of how partition pruning speeds up queries
- Familiarity with idempotent writes and the incremental-load idea ([Incremental Data Processing](../06-incremental-processing/) is a good warm-up)
- A columnar format or partitioned table target (Parquet, Delta/Iceberg/Hive-style, or a partitioned SQL table)

## Learning Objectives

By the end, you should be able to:

- Choose a partition key by query pattern and cardinality, not by habit
- Write data into the correct partition and overwrite a single partition idempotently
- Verify that a filtered query prunes to only the relevant partitions
- Manage a partition lifecycle: retention, compaction, and cleanup
- Backfill or repair one partition without disturbing the rest

## Functional Requirements

1. The pipeline must write output partitioned by a chosen key (e.g. event date).
2. Re-running for a given partition must fully overwrite that partition, not append duplicates.
3. A query filtered on the partition key must read only matching partitions (prunable layout).
4. The pipeline must support backfilling an arbitrary set of past partitions.
5. A retention policy must drop partitions older than a configured horizon.
6. The layout must avoid tiny-file explosion — control the number of files per partition.
7. The pipeline must record which partitions were written or rebuilt in each run.

## Suggested Milestones

1. **Milestone 1 — Partitioned write:** Pick a key and write output into per-partition paths.
2. **Milestone 2 — Idempotent overwrite & pruning:** Overwrite a single partition on rerun and confirm queries prune.
3. **Milestone 3 — Lifecycle:** Add backfill, retention, and file-count control.

## Data & Interface Sketch

```text
partition layout (date-partitioned):
  /warehouse/events/
    dt=2026-07-22/  part-000.parquet ...
    dt=2026-07-23/  part-000.parquet ...
    dt=2026-07-24/  part-000.parquet ...

write contract:
  write(partition=dt, rows) -> overwrite ONLY that dt's directory
  backfill(dt_from, dt_to)  -> rebuild each dt in range independently

pruning check:
  query WHERE dt = '2026-07-24'  -> reads 1 partition, not the full table

lifecycle:
  retention:   drop dt < today - N days
  compaction:  merge many small files -> few target-sized files
  run_manifest: run_id | partitions_written[] | files | rows
```

## Stretch Goals

- Handle skew: detect a hot partition and sub-partition it (e.g. by hash bucket) to balance load.
- Add multi-level partitioning (dt + region) and reason about the partition-explosion tradeoff.
- Track partition-level stats so a query planner can skip via min/max metadata, not just the key.
- Support late data by re-opening and rewriting an already-closed partition idempotently.

## Definition of Done

- [ ] Output lands in the correct partition path for its key.
- [ ] Reprocessing one date overwrites exactly that partition, leaving others untouched.
- [ ] A partition-filtered query demonstrably scans only the matching partitions.
- [ ] Backfilling a date range rebuilds each partition independently.
- [ ] Retention removes expired partitions and file counts stay bounded per partition.

## Common Pitfalls

- Choosing a high-cardinality partition key (e.g. user_id) and creating millions of tiny partitions.
- Appending on rerun so a reprocessed day silently doubles its rows.
- Partitioning on a column that queries never filter on, so pruning never kicks in.
- The small-files problem: thousands of KB-sized files per partition crushing read performance.
- Deleting a partition's files before the new write lands, leaving a gap on failure.

## Resources

- [Apache Hive: Partitioned tables](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-PartitionedTables) — the original directory-partition convention.
- [Databricks: Partitioning best practices](https://docs.databricks.com/aws/en/tables/partitions) — when to partition and the small-files trap.
- [Apache Iceberg: Partitioning](https://iceberg.apache.org/docs/latest/partitioning/) — hidden partitioning and evolution done right.
- [Spark: Data sources — partition discovery](https://spark.apache.org/docs/latest/sql-data-sources-parquet.html#partition-discovery) — how pruning uses the layout.
