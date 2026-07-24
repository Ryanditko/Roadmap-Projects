# Data Cleaning Pipeline (CSV)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Raw data is almost never analysis-ready: columns have inconsistent casing, numbers arrive as strings with currency symbols, dates come in five formats, and some rows are duplicated or half-empty. In this project you build a repeatable pipeline that takes a messy CSV and produces a clean, validated one — plus a short report describing exactly what was changed and why. The goal is not a one-off notebook full of manual fixes but a set of ordered, documented steps you can rerun on next month's export and get the same result.

## Prerequisites

- Basic Python and familiarity with a dataframe library (pandas or Polars)
- Understanding of common data types (string, integer, float, date, boolean)
- Ability to read a CSV and inspect its columns
- A messy real-world dataset — the [Kaggle Titanic dataset](https://www.kaggle.com/c/titanic/data) or any government open-data CSV works well because it has missing values and mixed types

## Learning Objectives

By the end, you should be able to:

- Profile a dataset to find missing values, duplicates, and type problems
- Choose and justify a missing-value strategy (drop, fill, interpolate) per column
- Detect outliers with a simple, defensible rule (IQR or z-score)
- Standardize categorical labels and normalize numeric ranges
- Express cleaning as ordered, reproducible steps rather than ad-hoc edits

## Functional Requirements

1. The pipeline must load a source CSV and report row count, column count, and dtype per column.
2. It must detect and remove exact duplicate rows, reporting how many were dropped.
3. Each column with missing values must have an explicitly chosen handling strategy, not a silent default.
4. Categorical values must be standardized (e.g. `"USA"`, `"usa"`, `" US "` collapse to one label).
5. At least one numeric column must be checked for outliers using a stated rule.
6. The pipeline must output a cleaned CSV plus a written summary of every transformation applied.
7. Running the pipeline twice on the same input must produce identical output.

## Suggested Milestones

1. **Milestone 1 — Profile:** Load the CSV and produce a quality report: shape, dtypes, null counts, duplicate count.
2. **Milestone 2 — Clean:** Apply duplicate removal, missing-value handling, and categorical standardization as discrete steps.
3. **Milestone 3 — Validate & report:** Add outlier and constraint checks, emit the cleaned file, and write a before/after summary.

## Data & Interface Sketch

```text
Cleaning report (per column)
  name:          string
  dtype:         inferred type
  null_count:    integer
  null_strategy: "drop" | "fill_mean" | "fill_mode" | "interpolate" | "keep"
  notes:         string

Pipeline stages (ordered, each rerunnable)
  1. load          -> raw dataframe + profile
  2. dedupe        -> removes exact-duplicate rows
  3. handle_nulls  -> applies per-column strategy
  4. standardize   -> trims, lowercases, maps categorical synonyms
  5. validate      -> range/constraint checks, outlier flags
  6. save          -> cleaned.csv + report.md

Rows in: N   Rows out: M   Duplicates removed: N-M
```

## Stretch Goals

- Add a config file (YAML/JSON) that declares per-column rules so the pipeline is data-driven, not hard-coded.
- Emit a data-quality score before and after to quantify improvement.
- Log rejected or quarantined rows to a separate file instead of dropping them silently.
- Add schema validation with a library like Pandera or Great Expectations.

## Definition of Done

- [ ] The pipeline turns the raw CSV into a cleaned CSV with no manual edits in between.
- [ ] Every missing-value decision is explicit and recorded in the report.
- [ ] Duplicate and outlier counts appear in the output summary.
- [ ] Categorical labels are consistent across the cleaned column.
- [ ] Running the pipeline twice yields verifiably equal output.

## Common Pitfalls

- Filling missing numeric values with the mean before removing outliers, which skews the mean.
- Dropping rows with any null and silently losing most of the dataset — check how much you discard.
- Standardizing categories with a manual list that breaks the moment a new label appears.
- Treating ID or ZIP-code columns as numbers, so leading zeros vanish and math is applied to them.

## Resources

- [pandas: Working with missing data](https://pandas.pydata.org/docs/user_guide/missing_data.html) — the canonical guide to `dropna`, `fillna`, and interpolation.
- [scikit-learn: Imputation of missing values](https://scikit-learn.org/stable/modules/impute.html) — strategies beyond simple fills.
- [Wikipedia: Interquartile range](https://en.wikipedia.org/wiki/Interquartile_range) — the IQR rule for outlier detection.
- [Great Expectations docs](https://docs.greatexpectations.io/) — declarative data validation if you take the stretch goal.
