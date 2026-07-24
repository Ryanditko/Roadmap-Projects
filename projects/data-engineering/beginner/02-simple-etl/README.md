# Simple ETL Pipeline

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build a small Extract-Transform-Load pipeline that reads records from one place, reshapes them, and writes them somewhere else. The classic beginner version: pull rows from a source file or table, clean and enrich them, and land them in a destination table. The value here is not any single step but the *shape* of the whole — three clearly separated stages joined by a well-defined record format. Once you can see extract, transform, and load as independent, testable functions, you have the backbone that every data pipeline, no matter how large, is built on.

## Prerequisites

- Comfort reading and writing files or a simple database (see [CSV to Database Loader](../01-csv-to-database/) first if that is new)
- Basic functions and data structures in your language of choice
- Understanding of what a record/row looks like as a dictionary or object
- Familiarity with running a script from the command line

## Learning Objectives

By the end, you should be able to:

- Separate a pipeline into distinct extract, transform, and load stages
- Model an in-flight record as a plain data structure passed between stages
- Apply field-level transformations: renaming, deriving, and type-casting
- Route bad records to a rejects sink instead of crashing the run
- Emit run metadata (counts, duration) so a pipeline is observable
- Design the transform stage to be pure and unit-testable

## Functional Requirements

1. The pipeline must have three separable stages, each callable independently for testing.
2. Extract must read from a defined source and yield records one at a time.
3. Transform must apply at least three operations: a rename, a derived field, and a type cast.
4. A record that fails transformation must be sent to a rejects sink with the reason, not halt the pipeline.
5. Load must write valid records to the destination and be safe to re-run.
6. The pipeline must print a summary: records extracted, loaded, and rejected, plus elapsed time.

## Suggested Milestones

1. **Milestone 1 — Extract & pass through:** Read the source and load unchanged records to the destination.
2. **Milestone 2 — Transform:** Insert the transform stage with renames, derived fields, and casts.
3. **Milestone 3 — Resilience & reporting:** Add the rejects sink and a run summary with counts and timing.

## Data & Interface Sketch

```text
extract (source: sales.csv)
  {"order":"A1","amount":"19.90","ts":"2024-03-01T10:00Z"}

transform
  order  -> order_id   (rename)
  amount -> amount_cents (float * 100 -> int; reject if not numeric)
  ts     -> order_date  (derive date from timestamp)

load (target: orders table)
  order_id | amount_cents | order_date

rejects sink -> rejects.jsonl
  {"record": {...}, "reason": "amount not numeric"}

flow: extract -> transform -> [ok] load
                           \-> [bad] rejects
summary: extracted=500 loaded=492 rejected=8 elapsed=1.4s
```

## Stretch Goals

- Add incremental extraction using a high-water mark (only new rows since last run).
- Make the transform config-driven so mappings live in a file, not code.
- Support multiple destinations (a table plus a Parquet file) from one run.
- Add a `--limit` flag to process a sample for quick iteration.

## Definition of Done

- [ ] Each stage can be called and tested in isolation.
- [ ] A valid source runs end-to-end and lands correct records.
- [ ] Bad records land in the rejects sink with a reason and do not stop the run.
- [ ] Re-running the pipeline does not duplicate loaded records.
- [ ] The summary reports extracted, loaded, and rejected counts plus timing.

## Common Pitfalls

- Fusing all three stages into one function, making anything untestable.
- Letting one bad record throw and kill the whole batch.
- Mutating source records in place instead of producing new transformed ones.
- Forgetting timezone or format normalization when deriving dates.

## Resources

- [Python Functional Programming HOWTO](https://docs.python.org/3/howto/functional.html) — generators and pure functions for pipeline stages.
- [Wikipedia: Extract, transform, load](https://en.wikipedia.org/wiki/Extract,_transform,_load) — the pattern and its vocabulary.
- [Apache Airflow concepts](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html) — how the industry orchestrates real ETL, worth reading for context.
- [petl documentation](https://petl.readthedocs.io/en/stable/) — a lightweight Python ETL library to compare your hand-rolled version against.
