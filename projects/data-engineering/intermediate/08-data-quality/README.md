# Data Quality Checks Framework

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a reusable framework that runs declarative quality checks against a dataset and reports where it falls short. Instead of scattering ad-hoc `assert` statements through every pipeline, you define rules once — "this column is never null", "order totals are non-negative", "customer_id is unique" — and let the framework evaluate them, score the dataset, and decide whether to warn or halt the pipeline. This is how teams stop shipping broken data downstream. The design challenge is making rules expressive enough to be useful, cheap enough to run on large tables, and structured enough that a failure tells you exactly which rows and which rule went wrong.

## Prerequisites

- Comfort querying or scanning a tabular dataset (SQL, a DataFrame API, or files)
- Understanding of common data quality dimensions: completeness, validity, uniqueness, consistency
- Familiarity with the idea of a declarative rule versus imperative code
- Any language and a dataset with realistic imperfections to test against

## Learning Objectives

By the end, you should be able to:

- Express quality rules declaratively and evaluate them uniformly
- Cover the core quality dimensions — completeness, validity, uniqueness, consistency, freshness
- Produce a structured report that names the rule, the affected rows, and the failure count
- Compute a dataset quality score and gate a pipeline on it
- Separate blocking failures from warnings so bad data does not always stop the line

## Functional Requirements

1. The framework must accept a set of rules bound to a dataset and columns, defined as data/config, not hardcoded logic.
2. It must support not-null, uniqueness, range/domain, regex/format, and cross-column consistency checks.
3. Each check must report pass/fail, the number of violating rows, and a sample of offending values.
4. It must compute an overall quality score (e.g. weighted pass rate) for the dataset.
5. Rules must carry a severity so the framework can warn versus block the pipeline.
6. A referential check must verify that keys in one dataset exist in another.
7. Results must be persisted per run so quality can be tracked over time.

## Suggested Milestones

1. **Milestone 1 — Rule engine:** Define a rule format and evaluate single-column checks, producing pass/fail results.
2. **Milestone 2 — Coverage & reporting:** Add cross-column, referential, and statistical checks with a structured report and score.
3. **Milestone 3 — Gating & history:** Add severities that gate the pipeline and persist results to track quality trends.

## Data & Interface Sketch

```text
rule
  id          string
  dataset     string
  column(s)   list
  type        enum(not_null | unique | range | regex | consistency | referential)
  params      map     (e.g. { min: 0 } or { pattern: "..." })
  severity    enum(warn | block)

check_result
  rule_id, passed(bool), rows_total, rows_failed, sample_values[], run_id, ran_at

report
  dataset, score(0..100), results[], blocking_failures(int)
  -> if blocking_failures > 0: fail the pipeline

example rule: { column: "email", type: regex, params: { pattern: name@example.com form } }
```

## Stretch Goals

- Add statistical checks: detect an outlier column mean or a distribution drift versus a baseline.
- Auto-suggest rules by profiling a clean sample (infer types, ranges, and null rates).
- Emit a per-run quality trend and alert when the score degrades between runs.
- Support quarantining failing rows to a side table instead of blocking the whole load.

## Definition of Done

- [ ] Rules are defined as configuration and run without touching framework code.
- [ ] A failing check reports the rule, the count, and a sample of the actual bad values.
- [ ] The quality score reflects the weighted outcome of all checks.
- [ ] A `block`-severity failure stops the pipeline; a `warn` failure lets it proceed.
- [ ] A referential check correctly catches an orphaned foreign key.

## Common Pitfalls

- Writing checks as one-off code so nothing is reusable across datasets — keep rules declarative.
- Reporting only pass/fail with no sample, forcing whoever triages to re-query the whole table.
- Running row-by-row in application code when a set-based query would be far cheaper at scale.
- Treating every failure as blocking, so a cosmetic issue halts a critical load.
- Checking uniqueness or nulls but never validating cross-column business rules where real bugs hide.

## Resources

- [Great Expectations: Core concepts](https://docs.greatexpectations.io/docs/core/introduction/) — a mature declarative data quality framework to study.
- [dbt: Tests](https://docs.getdbt.com/docs/build/data-tests) — how tests are declared and gated in a warehouse workflow.
- [Wikipedia: Data quality](https://en.wikipedia.org/wiki/Data_quality) — the standard quality dimensions.
- [Amazon Deequ paper](https://www.vldb.org/pvldb/vol11/p1781-schelter.pdf) — automating large-scale data quality verification.
