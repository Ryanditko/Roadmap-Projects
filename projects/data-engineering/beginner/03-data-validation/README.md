# Data Validation Script

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Before data can be trusted, it has to be checked. Build a script that takes a dataset and a set of rules — "this column is required," "this must be a positive number," "emails must look like emails" — and produces a report of exactly which rows broke which rule. This is the guardrail that sits at the front door of every serious pipeline: it turns "the data looks weird" into a precise, line-by-line diagnosis. The interesting design work is not the checks themselves but how you express rules cleanly and how you report failures so a human can actually act on them.

## Prerequisites

- Ability to read a CSV or JSON dataset (see [CSV to Database Loader](../01-csv-to-database/) if new)
- Comfort with conditionals, loops, and basic string handling
- Familiarity with regular expressions at a beginner level
- Understanding of data types: string, number, date, boolean

## Learning Objectives

By the end, you should be able to:

- Express validation rules as data or small functions rather than tangled `if` blocks
- Distinguish structural checks (type, required) from semantic checks (range, format)
- Collect *all* violations for a row instead of stopping at the first
- Produce a human-readable report and a machine-readable one
- Compute quality metrics like completeness and validity rates per column
- Exit with a status code that a scheduler or CI job can react to

## Functional Requirements

1. The script must accept a dataset and a rule set describing per-column constraints.
2. It must support at least: required/NOT NULL, type check, numeric range, and regex format.
3. Every row must be checked against all applicable rules, collecting every failure, not just the first.
4. The output must include a summary (rows checked, rows with errors, error count per rule) and per-row detail.
5. A machine-readable report (JSON or CSV of violations) must be written for downstream use.
6. The process must exit non-zero when any rule fails, so automation can gate on it.

## Suggested Milestones

1. **Milestone 1 — One rule:** Check a single required-column rule and print failing row numbers.
2. **Milestone 2 — Rule set:** Support multiple rule types read from a config and collect all violations per row.
3. **Milestone 3 — Reporting:** Emit a summary, a violations file, quality metrics, and a proper exit code.

## Data & Interface Sketch

```text
rule set (config)
  age    -> {required: true, type: int, min: 0, max: 120}
  email  -> {required: true, regex: "^[^@]+@[^@]+\\.[^@]+$"}
  status -> {allowed: ["active","churned","trial"]}

input row (line 7)
  {"age": "-3", "email": "bob@", "status": "active"}

violations collected
  line 7 age   -> below min (0)
  line 7 email -> fails regex

report summary
  rows=1000 clean=944 with_errors=56
  by_rule: age.min=12 email.regex=31 status.allowed=13
  metrics: email.validity=96.9% age.completeness=99.1%

exit code: 1 (violations present)
```

## Stretch Goals

- Add cross-field rules (e.g. `end_date` must be after `start_date`).
- Support uniqueness checks across the whole dataset (detect duplicate keys).
- Add simple statistical anomaly flags (values beyond N standard deviations).
- Make rule severity configurable (error vs warning) and only fail on errors.

## Definition of Done

- [ ] Rules are defined as configuration, not hard-coded per dataset.
- [ ] Every failing row lists all of its violations, not just one.
- [ ] Both a human summary and a machine-readable violations file are produced.
- [ ] Quality metrics per column are reported.
- [ ] The script exits non-zero exactly when violations exist.

## Common Pitfalls

- Stopping at the first error in a row, hiding the other problems from the user.
- Treating an empty string, `null`, and a missing key as the same without deciding intentionally.
- Regexes that are too strict (rejecting valid emails) or too loose (accepting junk).
- Reporting only counts without line numbers, so nobody can find the bad rows.

## Resources

- [Great Expectations docs](https://docs.greatexpectations.io/docs/home/) — the reference vocabulary for data validation "expectations".
- [Pandera documentation](https://pandera.readthedocs.io/en/stable/) — schema and statistical validation for dataframes.
- [MDN: Regular expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions) — a solid, practical regex primer.
- [JSON Schema](https://json-schema.org/learn/getting-started-step-by-step) — a standard way to express structural validation rules.
