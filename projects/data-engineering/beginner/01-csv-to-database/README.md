# CSV to Database Loader

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Take a raw CSV file — the kind exported from a spreadsheet or a legacy system — and load it cleanly into a relational database table. Along the way you will confront the questions every ingestion job eventually asks: what type is each column, what do I do with the row that has a letter where a number belongs, and how do I re-run this tomorrow without doubling every record? This project keeps the scope small (one file, one table) so you can focus on doing the load *correctly* rather than *fast*, and it gives you a reusable mental model for the CSV-to-warehouse pattern that shows up everywhere in data work.

## Prerequisites

- Basic Python (or a language with a CSV library and a DB driver)
- A relational database you can run locally (SQLite needs no setup; Postgres is a good next step)
- Comfort writing simple `CREATE TABLE` and `INSERT` statements
- Understanding of what a primary key is and why it matters

## Learning Objectives

By the end, you should be able to:

- Stream a CSV file row by row instead of loading it all into memory
- Infer or declare a column type and coerce string values into it
- Design a table schema that matches your source data
- Perform inserts in batches inside a transaction
- Skip or quarantine malformed rows without aborting the whole load
- Make the loader idempotent so a re-run does not create duplicates

## Functional Requirements

1. The tool must read a CSV with a header row and map each column to a table field by name, not position.
2. The tool must create the target table if it does not already exist.
3. Each value must be coerced to its declared type; a row that fails coercion must be rejected, logged with its line number, and skipped — never silently dropped.
4. Rows must be inserted inside a transaction so a mid-load failure leaves no partial batch behind.
5. Re-running the loader on the same file must not create duplicate rows (use a natural or primary key).
6. On completion, the tool must report counts: rows read, inserted, and rejected.

## Suggested Milestones

1. **Milestone 1 — Parse & print:** Read the CSV and print typed rows to the console; no database yet.
2. **Milestone 2 — Load:** Create the table and insert rows in batched transactions.
3. **Milestone 3 — Robustness:** Add type coercion with a rejects log, idempotent re-runs, and a summary report.

## Data & Interface Sketch

```text
source: users.csv
  id,name,signup_date,score
  1,Ana,2024-01-05,88

transform:
  id          -> INTEGER   (reject if not parseable)
  name        -> TEXT      (trim whitespace)
  signup_date -> DATE      (parse ISO-8601)
  score       -> REAL      (NULL if blank)

target table: users (id PRIMARY KEY, name, signup_date, score)

rejects.log: line 42: score="N/A" not a number -> skipped

CLI: loader --file users.csv --table users --batch 500
summary: read=1000 inserted=987 rejected=13
```

## Stretch Goals

- Infer column types automatically by sampling the first N rows.
- Support "upsert" so existing keys are updated instead of skipped.
- Add a `--dry-run` flag that validates without writing.
- Stream a gzipped CSV without fully decompressing to disk.

## Definition of Done

- [ ] A well-formed CSV loads fully into the table with correct types.
- [ ] Malformed rows are logged with line numbers and skipped.
- [ ] Running the loader twice yields the same row count as once.
- [ ] A failure mid-load rolls back the current transaction cleanly.
- [ ] The final summary reports read, inserted, and rejected counts.

## Common Pitfalls

- Reading the whole file into memory — use a streaming reader so large files do not exhaust RAM.
- Inserting one row per statement, which is slow; batch inside a transaction instead.
- Treating empty strings as valid numbers or dates; decide on NULL handling explicitly.
- Assuming column positions never change instead of mapping by header name.

## Resources

- [Python `csv` module](https://docs.python.org/3/library/csv.html) — the standard streaming CSV reader.
- [SQLite documentation](https://www.sqlite.org/docs.html) — zero-setup database ideal for this project.
- [PostgreSQL `COPY`](https://www.postgresql.org/docs/current/sql-copy.html) — the fast bulk-load path once you outgrow row inserts.
- [pandas `read_csv`](https://pandas.pydata.org/docs/reference/api/pandas.read_csv.html) — a higher-level option worth comparing against manual parsing.
