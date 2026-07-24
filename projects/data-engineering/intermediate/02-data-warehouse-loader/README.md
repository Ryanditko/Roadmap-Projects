# Data Warehouse Loader

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build the loading layer that turns raw operational records into a query-friendly dimensional model in a warehouse. You will land source data into a staging area, then load it into fact and dimension tables shaped as a star schema. The centerpiece is handling *slowly changing dimensions* (SCD Type 2): when a customer changes their city, you keep the old row, close it out, and open a new versioned row — so historical facts still join to the address that was true at the time. You will also make the load incremental and idempotent, so re-running a batch corrects itself instead of doubling revenue, and support a backfill that rebuilds a range of past batches. This is the difference between a warehouse people trust and a spreadsheet with extra steps.

## Prerequisites

- Solid SQL: joins, `GROUP BY`, window functions, and `MERGE`/upsert semantics
- Understanding of primary/foreign keys and referential integrity
- Familiarity with a warehouse or database (Postgres, DuckDB, BigQuery, or Snowflake)
- Basic grasp of the ETL/ELT distinction
- The [Airflow ETL project](../01-airflow-etl/) is a useful warm-up for the load orchestration

## Learning Objectives

By the end, you should be able to:

- Design a star schema with fact and dimension tables and surrogate keys
- Implement SCD Type 2 so dimension history is preserved and queryable as-of a date
- Load incrementally using a high-water mark rather than reprocessing everything
- Make loads idempotent via upsert/merge so re-runs never double-count
- Backfill a range of batches and verify referential integrity afterward

## Functional Requirements

1. Source data must first land in a staging table before any transformation into the model.
2. The warehouse must expose at least one fact table and two dimensions in a star schema, keyed by surrogate keys.
3. A tracked dimension must implement SCD Type 2 with `valid_from`, `valid_to`, and a current-row flag.
4. Loads must be incremental, selecting only rows changed since the last successful high-water mark.
5. Re-running a batch must be idempotent — an upsert/merge, not a blind insert.
6. Fact rows must reference the dimension version that was current at the event's timestamp.
7. A backfill must be able to rebuild a specified range of batches without duplicating rows.

## Suggested Milestones

1. **Milestone 1 — Stage & load dims:** Land source into staging and populate dimensions with surrogate keys.
2. **Milestone 2 — SCD Type 2 & facts:** Version a changing dimension and load facts that join to the correct version.
3. **Milestone 3 — Incremental & idempotent:** Add a high-water mark, make the merge idempotent, and test a backfill.

## Data & Interface Sketch

```text
staging_customer(raw cols..., loaded_at)

dim_customer                          fact_orders
  customer_sk   PK (surrogate)          order_sk     PK
  customer_id   (natural/business key)  customer_sk  FK -> dim_customer
  city                                  order_date_sk FK -> dim_date
  valid_from    timestamp               amount
  valid_to      timestamp (or NULL)     ...
  is_current    boolean

SCD2 on change of city:
  UPDATE dim_customer SET valid_to = now(), is_current = false
    WHERE customer_id = ? AND is_current;
  INSERT new row with new city, valid_from = now(), is_current = true;

Incremental: WHERE source.updated_at > last_high_water_mark
Idempotent load: MERGE ... ON natural_key WHEN MATCHED ... WHEN NOT MATCHED ...
```

## Stretch Goals

- Add SCD Type 1 for an attribute where history is not worth keeping, and contrast the two.
- Build an as-of query that reconstructs the dimension state on an arbitrary past date.
- Add a late-arriving-fact handler that back-dates to the correct dimension version.
- Track load metrics (rows inserted/updated/rejected) per batch for monitoring.

## Definition of Done

- [ ] Facts join to dimensions with zero orphan surrogate keys (referential integrity holds).
- [ ] Changing a tracked attribute closes the old dimension row and opens a new current one.
- [ ] Re-running the same batch leaves row counts unchanged (idempotent merge verified).
- [ ] An incremental run touches only rows changed since the last high-water mark.
- [ ] A backfill over several batches reproduces the same result as loading them in order.

## Common Pitfalls

- Using the natural business key as the fact foreign key, which breaks the moment SCD2 creates a second version.
- Forgetting to close the previous row's `valid_to`, leaving two "current" rows for one entity.
- Blind `INSERT` loads that double data on re-run instead of `MERGE`/upsert.
- Advancing the high-water mark before the load commits, so a mid-load crash skips rows forever.
- Overlapping `valid_from`/`valid_to` ranges that make as-of joins return duplicates.

## Resources

- [Kimball Group: Dimensional Modeling Techniques](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/) — the canonical reference for star schemas and SCDs.
- [Wikipedia: Slowly Changing Dimension](https://en.wikipedia.org/wiki/Slowly_changing_dimension) — a concise tour of SCD types 1–6.
- [dbt: SCD / snapshots](https://docs.getdbt.com/docs/build/snapshots) — how a modern tool models Type 2 history.
- [Star schema (Wikipedia)](https://en.wikipedia.org/wiki/Star_schema) — facts, dimensions, and surrogate keys.
