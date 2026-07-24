# Scalable Data Lake Architecture

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Design a scalable data lake that behaves like a warehouse where it matters — a **lakehouse**. Raw files in object storage are cheap and infinite but query slowly and can't do atomic updates; a table format (Apache Iceberg, Delta Lake, or Apache Hudi) sits on top and adds ACID commits, schema evolution, time travel, and file compaction. You will design the storage layout (hot/warm/cold tiering, partitioning, file sizing), pick a table format and justify it, and prove that a well-organized lake answers analytical queries fast while a naive dump of small files does not. The theme running through it is that layout *is* performance: partition pruning, file compaction, and metadata pruning are what separate a query that scans a terabyte from one that scans a gigabyte.

## Prerequisites

- Comfort with object storage (S3/GCS/Azure Blob) and a columnar format (Parquet/ORC)
- A query engine you can point at files (Spark, Trino, DuckDB, or Athena)
- Understanding of partitioning and predicate pushdown
- Familiarity with the "small files problem" and why it hurts

## Learning Objectives

By the end, you should be able to:

- Choose a table format (Iceberg / Delta / Hudi) and justify it against your workload
- Design partitioning and file sizing so queries prune instead of full-scan
- Implement tiering (hot/warm/cold) with lifecycle rules and reason about cost/latency tradeoffs
- Use ACID table operations: atomic append, schema evolution, and time travel
- Measure query cost (bytes scanned, latency) and tie it back to layout decisions

## Functional Requirements

1. The lake must store data in a table format providing atomic commits and schema evolution.
2. Data must be partitioned so a filtered query prunes partitions rather than scanning everything.
3. A compaction process must consolidate small files into target-sized files without downtime for readers.
4. The design must define tiering with a documented lifecycle policy (e.g. cold data → cheaper storage class after N days).
5. The table must support time travel: a query must be able to read a previous snapshot.
6. Query cost (bytes scanned, latency) must be measurable and reported before and after optimization.

## Suggested Milestones

1. **Milestone 1 — Table format on object storage:** Land data in an Iceberg/Delta/Hudi table with a partition scheme; run a baseline query.
2. **Milestone 2 — Optimize layout:** Add compaction and right-size files; measure the drop in bytes scanned and latency.
3. **Milestone 3 — Tiering & time travel:** Add lifecycle tiering and demonstrate reading a prior snapshot after a schema change.

## Data & Interface Sketch

```text
[ingest] ─▶ object store bucket
   warehouse/db/events/
     metadata/           <- table format: snapshots, schema, manifest list
     data/dt=2026-07-24/ part-0001.parquet (target ~128-512MB each)
         dt=2026-07-23/ ...

query: SELECT ... WHERE dt = '2026-07-24' AND region = 'us'
   -> partition prune (dt) -> file prune via column stats (region)
   -> scans a few files, not the table

compaction job: many small files -> fewer right-sized files (atomic rewrite)
time travel:   SELECT ... FOR SYSTEM_VERSION AS OF <snapshotId>

Tiering: hot (last 7d, standard) | warm (30d, infrequent) | cold (>90d, archive)
Non-functional: query p95 < T, storage cost/TB < $C, no reader downtime on compaction.
```

## Stretch Goals

- Add hidden/transform partitioning (e.g. Iceberg's `days(ts)`) and compare pruning against manual partition columns.
- Implement a Z-order or clustering step and measure its effect on multi-column filters.
- Add a metadata/statistics-driven query cost estimate and validate it against actual bytes scanned.

## Definition of Done

- [ ] Data is stored in a table format with atomic commits and evolvable schema.
- [ ] A filtered query demonstrably prunes partitions/files instead of full-scanning.
- [ ] Compaction reduces file count and improves query latency without breaking concurrent reads.
- [ ] Time travel returns a correct earlier snapshot after a schema change.
- [ ] A before/after benchmark documents bytes scanned, latency, and storage cost across tiers.

## Common Pitfalls

- The small files problem: streaming or over-partitioning produces thousands of tiny files that murder query planning.
- Partitioning on a high-cardinality column, creating millions of partitions and slow metadata.
- Treating the lake as a filesystem and losing atomicity — half-written data read as complete.
- Ignoring metadata growth in the table format; unbounded snapshots need expiration too.

## Resources

- [Apache Iceberg documentation](https://iceberg.apache.org/docs/latest/) — table spec, partitioning, and snapshots.
- [Delta Lake documentation](https://docs.delta.io/latest/index.html) — ACID transactions and time travel on the lake.
- [Apache Hudi documentation](https://hudi.apache.org/docs/overview) — copy-on-write vs merge-on-read tradeoffs.
- [Trino: Object storage & pruning](https://trino.io/docs/current/connector/hive.html) — how a query engine prunes lake data.
