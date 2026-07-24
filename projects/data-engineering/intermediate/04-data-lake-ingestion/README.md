# Data Lake Ingestion System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build an ingestion system that lands raw data from several formats into a data lake and registers it in a catalog so it can actually be found and queried. The lake itself is just object storage (local filesystem, MinIO, or S3), but a lake without organization is a swamp. Your job is the layer that gives it structure: a zoned layout (raw → cleaned), consistent partitioning, extracted metadata, and a catalog that records what landed, when, and in what schema. You will ingest CSV, JSON, and Parquet, normalize them into a columnar format, and make ingestion idempotent so re-running a source drop replaces rather than duplicates. The payoff is understanding why "just dump the files in S3" is where data engineering begins, not ends.

## Prerequisites

- Comfort reading and writing files in a language like Python
- Familiarity with columnar formats (Parquet) and row formats (CSV/JSON)
- Basic understanding of object storage and key/prefix layouts
- Access to local object storage (filesystem, MinIO, or a cloud bucket)

## Learning Objectives

By the end, you should be able to:

- Design a zoned lake layout (raw vs cleaned) with a consistent partition scheme
- Ingest heterogeneous formats and normalize them to a single columnar format
- Extract and record metadata (schema, row count, source, ingest time) in a catalog
- Make ingestion idempotent so re-dropping a file does not duplicate data
- Support a backfill that re-ingests a range of historical source drops

## Functional Requirements

1. The system must ingest at least three formats (e.g. CSV, JSON, Parquet) through a common interface.
2. Raw files must land unchanged in a raw zone, then be normalized into a cleaned zone as Parquet.
3. Data must be partitioned by a stable key (e.g. source and ingest date) using a consistent path layout.
4. Each ingested dataset must be registered in a catalog with schema, row count, source, and timestamp.
5. Re-ingesting the same source drop must be idempotent — it replaces the partition, not appends to it.
6. The system must support backfilling a date range of source drops without manual per-file steps.
7. An ingestion failure must leave the target partition in its prior consistent state, never half-written.

## Suggested Milestones

1. **Milestone 1 — Land raw:** Ingest one format into a partitioned raw zone with correct paths.
2. **Milestone 2 — Normalize & catalog:** Convert to Parquet in the cleaned zone and register metadata in the catalog.
3. **Milestone 3 — Idempotency & backfill:** Make partition writes atomic/idempotent and re-ingest a historical range.

## Data & Interface Sketch

```text
lake/
  raw/     source=orders/ingest_date=2024-01-01/orders.csv
  cleaned/ source=orders/ingest_date=2024-01-01/part-000.parquet

catalog entry:
  dataset:      "orders"
  path:         "cleaned/source=orders/ingest_date=2024-01-01/"
  format:       "parquet"
  schema:       [ {name, type}, ... ]
  row_count:    12045
  source_uri:   "sftp://.../orders.csv"
  ingested_at:  "2024-01-01T02:15:00Z"

ingest(source, ingest_date):
  write to <tmp>/... ; on success atomically swap into partition (idempotent)
Backfill: for d in date_range: ingest("orders", d)
```

## Stretch Goals

- Add schema inference plus a check that fails ingestion when a file's schema drifts unexpectedly.
- Track simple lineage: which raw file produced which cleaned partition.
- Add an open table format (Delta Lake, Apache Iceberg, or Hudi) and compare it to plain Parquet + catalog.
- Compress and compact many small files into fewer right-sized ones.

## Definition of Done

- [ ] All three formats ingest through one interface and land as Parquet in the cleaned zone.
- [ ] Partitions follow a consistent, queryable path layout keyed by source and date.
- [ ] The catalog reflects every dataset's schema, row count, source, and ingest time.
- [ ] Re-ingesting the same drop leaves row counts unchanged (idempotent partition swap).
- [ ] A backfill over several dates populates each partition and its catalog entry.

## Common Pitfalls

- Writing directly into the final partition path, so a crash leaves a half-written, unreadable partition.
- Appending to a partition on re-ingest instead of replacing it, silently duplicating rows.
- Letting each source pick its own path convention, making the lake unqueryable as a whole.
- Producing thousands of tiny files, which cripples downstream query engines.
- Storing metadata only in file names instead of a real catalog, so discovery requires listing everything.

## Resources

- [AWS: What is a data lake?](https://aws.amazon.com/what-is/data-lake/) — zones, catalogs, and lake architecture.
- [Apache Parquet documentation](https://parquet.apache.org/docs/) — the columnar format and why it suits lakes.
- [Databricks: Medallion architecture](https://www.databricks.com/glossary/medallion-architecture) — raw/bronze → silver → gold zoning.
- [Apache Iceberg documentation](https://iceberg.apache.org/docs/latest/) — an open table format for lakes.
