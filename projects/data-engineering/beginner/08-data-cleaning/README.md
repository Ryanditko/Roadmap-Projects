# Data Cleaning Pipeline

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Real-world data is messy: the same customer spelled three ways, dates in five formats, phone numbers with and without country codes, blank cells that mean different things. Build a pipeline that takes a dirty dataset and produces a clean, normalized one — deduplicating records, filling or flagging missing values, and standardizing formats. The discipline here is to make every cleaning decision *explicit and reversible*: you keep an audit trail of what changed and why, so a downstream analyst can trust the output and trace any value back to its raw form.

## Prerequisites

- Ability to read and write a CSV or JSON dataset
- Comfort with string manipulation and basic data types
- Understanding of what "duplicate" and "missing" mean for your data
- Familiarity with dictionaries/maps for grouping records

## Learning Objectives

By the end, you should be able to:

- Detect duplicate records by an exact key and by a normalized/fuzzy key
- Choose and apply a missing-value strategy (drop, default, or flag) per column
- Normalize formats: trim, case, dates, and numeric units to a canonical form
- Keep an audit trail recording every transformation applied to each record
- Produce before/after quality metrics so cleaning is measurable
- Separate "cleaned" from "unfixable" records rather than forcing every row through

## Functional Requirements

1. The pipeline must remove exact-duplicate rows and detect near-duplicates by a normalized key.
2. It must apply a configurable missing-value strategy per column, never guessing silently.
3. It must normalize at least: whitespace, letter case, date formats, and one numeric/unit field.
4. Every change must be recorded in an audit trail tying the cleaned value to its original.
5. Records that cannot be cleaned to meet the rules must be routed to a rejects output, not forced through.
6. The pipeline must report quality metrics before and after: completeness, duplicate rate, and rows rejected.

## Suggested Milestones

1. **Milestone 1 — Normalize:** Apply whitespace, case, and format normalization to each field.
2. **Milestone 2 — Dedup & missing:** Remove duplicates and apply per-column missing-value strategies.
3. **Milestone 3 — Audit & metrics:** Add the audit trail, rejects routing, and before/after quality metrics.

## Data & Interface Sketch

```text
source row (dirty)
  { "name": "  ANA souza ", "email": "ANA@example.com",
    "phone": "(81) 9999-1111", "signup": "01/05/2024" }

cleaning steps
  name   -> trim + title-case        -> "Ana Souza"
  email  -> trim + lower-case        -> "ana@example.com"
  phone  -> strip non-digits + E.164 -> "+558199991111"
  signup -> parse dd/mm/yyyy -> ISO  -> "2024-05-01"

dedup key: lower(email)   -> collapse duplicates, keep newest

audit trail (per record)
  { id: 7, changes: ["name:trim+case", "signup:reformatted"] }

rejects -> unfixable.jsonl  (e.g. phone has no digits)

metrics
  before: rows=1000 completeness=82% duplicate_rate=6%
  after:  rows=940  completeness=99% duplicate_rate=0% rejected=12
```

## Stretch Goals

- Add fuzzy matching (edit distance) to catch near-duplicate names, not just exact keys.
- Make cleaning rules config-driven so the same engine handles different datasets.
- Support "soft" vs "hard" cleaning modes (flag-only vs modify-in-place).
- Emit a diff report showing sample before/after values for spot-checking.

## Definition of Done

- [ ] Exact and near-duplicate records are collapsed by the chosen key.
- [ ] Each column's missing-value strategy is applied explicitly, not by accident.
- [ ] Whitespace, case, dates, and the numeric field are normalized consistently.
- [ ] Every changed value is traceable to its original via the audit trail.
- [ ] Before/after quality metrics and a rejects count are reported.

## Common Pitfalls

- Filling missing values with a default that pollutes analysis (e.g. `0` where NULL means "unknown").
- Deduplicating on the raw key so "Ana" and "ANA " survive as two records.
- Applying an irreversible transform with no record of the original value.
- Forcing unfixable rows through instead of quarantining them, corrupting the clean set.

## Resources

- [pandas: Working with missing data](https://pandas.pydata.org/docs/user_guide/missing_data.html) — strategies for nulls done well.
- [E.164 phone number format](https://en.wikipedia.org/wiki/E.164) — the canonical international phone standard.
- [Unicode normalization forms](https://unicode.org/reports/tr15/) — why text needs canonicalizing before comparison.
- [Tidy Data (Hadley Wickham)](https://vita.had.co.nz/papers/tidy-data.pdf) — the paper defining what "clean" tabular data means.
