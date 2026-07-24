# Dataset Comparison Tool

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

When a data pipeline breaks silently, the symptom is usually a dataset that quietly changed shape: a column dropped, a distribution shifted, nulls crept in. In this project you build a tool that takes two datasets — think "last month vs this month" or "training data vs production data" — and reports exactly how they differ. It profiles each one, compares schemas and statistics side by side, flags distribution drift, and produces a readable report. This is the foundation of data monitoring and validation, and it teaches you to describe a dataset precisely enough to notice when it changes.

## Prerequisites

- Basic Python and pandas
- Basic descriptive statistics (mean, median, standard deviation, quantiles)
- Understanding of a dataset's schema (column names and types)
- Two related datasets to compare — two snapshots of the same [Kaggle](https://www.kaggle.com/datasets) dataset over time, or a dataset you deliberately split and perturb

## Learning Objectives

By the end, you should be able to:

- Profile a dataset into a compact, comparable summary
- Compare two schemas and detect added, removed, or retyped columns
- Compare distributions of shared columns and quantify drift
- Choose a simple statistical test to check whether two samples differ
- Generate a clear side-by-side comparison report

## Functional Requirements

1. The tool must load two datasets and profile each (shape, columns, dtypes, null rates).
2. It must report schema differences: columns only in A, only in B, and type mismatches.
3. For shared numeric columns, it must compare summary statistics side by side.
4. It must flag columns whose distribution has shifted beyond a chosen threshold.
5. It must apply at least one statistical test (e.g. Kolmogorov–Smirnov or chi-square) to a shared column.
6. It must handle datasets with partially overlapping columns without crashing.
7. It must output a single readable comparison report.

## Suggested Milestones

1. **Milestone 1 — Profile:** Build a profiling summary for each dataset independently.
2. **Milestone 2 — Compare:** Diff the schemas and put shared-column statistics side by side.
3. **Milestone 3 — Detect drift & report:** Add a distribution test, flag drift, and assemble the report.

## Data & Interface Sketch

```text
Per-dataset profile
  rows, cols
  per column: { name, dtype, null_rate, n_unique, mean?, std?, min?, max? }

Comparison output
  schema diff
    only_in_A:   [ ... ]
    only_in_B:   [ ... ]
    type_change: [ { col, type_A, type_B } ]
  shared numeric columns (side by side)
    col        mean_A   mean_B   std_A   std_B   null_A   null_B   drift?
    age        38.1     41.7     11.2    12.9    0.0%     3.4%     YES
  distribution test
    KS(col) -> statistic, p_value -> "distributions differ" if p < 0.05
```

## Stretch Goals

- Add a numeric "difference score" per column and rank columns by how much they changed.
- Visualize overlaid distributions for the most-drifted columns.
- Compare categorical columns by value-frequency shift, not just presence.
- Wrap the tool so it can run on a schedule and alert when drift exceeds a threshold.

## Definition of Done

- [ ] Both datasets are profiled with matching summary fields.
- [ ] Schema differences (added, removed, retyped columns) are reported.
- [ ] Shared numeric columns are compared statistic-by-statistic.
- [ ] At least one distribution test is applied and its result interpreted.
- [ ] The tool runs on datasets with only partially overlapping columns.

## Common Pitfalls

- Assuming both datasets share every column, then crashing on the first mismatch.
- Comparing means only, missing a variance or shape change that leaves the mean intact.
- Reading a statistical test's p-value as "how big" the difference is (it is not).
- Treating tiny sample-size differences as drift when they are just noise.

## Resources

- [pandas: DataFrame.describe](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.describe.html) — the quick statistical profile.
- [SciPy: Kolmogorov–Smirnov test](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.ks_2samp.html) — comparing two continuous distributions.
- [SciPy: Chi-square test](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.chi2_contingency.html) — comparing categorical distributions.
- [Evidently AI: Data drift](https://docs.evidentlyai.com/) — how production drift monitoring formalizes this idea.
