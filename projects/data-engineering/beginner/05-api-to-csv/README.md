# API to CSV Exporter

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

A huge amount of data lives behind HTTP APIs, one page at a time. Build a tool that calls a paginated REST API, walks through every page, flattens each JSON record into a flat row, and writes the whole result to a clean CSV. The real lessons hide in the corners: following pagination correctly so you get *all* the data and no duplicates, backing off when the API rate-limits you, and retrying a flaky request without corrupting your output. It is the first project where your program depends on a system you do not control, so handling its failures gracefully is the whole point.

## Prerequisites

- Ability to make an HTTP GET request in your language of choice
- Understanding of JSON structure (objects, arrays, nesting)
- Comfort writing a CSV file (see [CSV to Database Loader](../01-csv-to-database/) for CSV basics)
- Awareness of HTTP status codes, especially 429 and 5xx

## Learning Objectives

By the end, you should be able to:

- Consume a paginated API using cursor- or offset-based pagination
- Respect rate limits and back off on `429 Too Many Requests`
- Retry transient failures with exponential backoff and a cap
- Flatten nested JSON into columns for a tabular CSV
- Map API fields to CSV headers via configuration, not hard-coding
- Deduplicate records by a stable key across pages

## Functional Requirements

1. The tool must fetch all pages of a paginated endpoint until there are no more.
2. It must handle `429` and `5xx` responses by retrying with backoff, up to a configurable limit.
3. It must transform each JSON record into a flat row using a defined field mapping.
4. Nested fields must be flattened or extracted; arrays must have a documented handling (join, first, or count).
5. Duplicate records (same key appearing across pages) must be written only once.
6. The output must be a valid CSV with a header row and a run summary of pages fetched and rows written.

## Suggested Milestones

1. **Milestone 1 — Fetch one page:** Call the endpoint once and print the parsed records.
2. **Milestone 2 — Paginate:** Follow pagination to the end, accumulating records with retry/backoff.
3. **Milestone 3 — Transform & export:** Flatten records, deduplicate, and write the CSV with a summary.

## Data & Interface Sketch

```text
GET /api/v1/users?page=1  -> { "data": [ ... ], "next": "?page=2" }

api record (nested)
  { "id": 7, "name": "Ana",
    "address": { "city": "Recife" },
    "tags": ["vip","beta"] }

field mapping (config)
  id            -> id
  name          -> name
  address.city  -> city         (flatten nested)
  tags          -> tags         (join with ";")

csv row
  id,name,city,tags
  7,Ana,Recife,vip;beta

retry policy
  429/5xx -> wait (2^attempt * base), max 5 attempts, honor Retry-After

summary: pages=12 rows=1180 duplicates_skipped=4 retries=3
```

## Stretch Goals

- Support incremental export using an `updated_since` parameter and a saved high-water mark.
- Stream rows to the CSV as pages arrive instead of buffering everything in memory.
- Add a config-driven authentication header (API key or bearer token) read from the environment.
- Emit both CSV and newline-delimited JSON from the same run.

## Definition of Done

- [ ] Every page is fetched; the final row count matches the API's reported total.
- [ ] Rate-limit and server errors trigger backoff and eventual success or a clear failure.
- [ ] Nested fields are flattened correctly and arrays handled per the documented rule.
- [ ] No duplicate keys appear in the output CSV.
- [ ] A summary reports pages fetched, rows written, and retries.

## Common Pitfalls

- Assuming a fixed number of pages instead of following the API's "next" signal.
- Hammering the API and getting throttled because you ignore `429` and `Retry-After`.
- Losing nested data by writing a Python dict or JSON blob straight into a CSV cell.
- Buffering all records in memory for a large dataset instead of streaming to disk.

## Resources

- [MDN: HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) — what 429, 500, and 503 mean for retry logic.
- [Google Cloud: Retry with exponential backoff](https://cloud.google.com/storage/docs/retry-strategy) — a clear, standard backoff strategy.
- [Python `requests`](https://requests.readthedocs.io/en/latest/) — the ergonomic HTTP client for the fetch layer.
- [REST API pagination patterns](https://developer.mozilla.org/en-US/docs/Web/HTTP/Link) — how APIs signal the next page via headers.
