# Multi-Source Ingestion Pipeline

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a pipeline that pulls the same kind of entity — say, customers or products — from several very different sources and merges them into one clean, consistent dataset. One source is a REST API, another a CSV drop, another a database export; each names its fields differently, disagrees on formats, and sometimes describes the same real-world record. Your job is to hide that mess behind a common connector interface, map every source into a shared schema, and resolve conflicts when two sources disagree. This is the integration reality behind every "single view of the customer" project, and the lesson is that the hard part is never the fetching — it is reconciliation.

## Prerequisites

- Comfort calling an HTTP API and reading files (CSV/JSON) programmatically
- Understanding of schema mapping and normalization
- Familiarity with deduplication and the idea of a natural/business key
- Any language, plus two or three sources (real or mocked) describing overlapping records

## Learning Objectives

By the end, you should be able to:

- Design a connector interface that abstracts over heterogeneous sources
- Map each source's fields and formats into one shared target schema
- Deduplicate records that represent the same entity across sources
- Resolve conflicts with an explicit precedence or recency strategy
- Isolate failures so one broken source does not sink the whole run

## Functional Requirements

1. Each source must be accessed through a common connector interface (fetch → raw records).
2. Every source must map into a single shared schema with normalized types and formats.
3. The pipeline must deduplicate records across sources using a defined natural key.
4. When sources disagree on a field, a documented conflict-resolution rule must decide the winner.
5. A failure in one source must not abort ingestion of the others; it must be recorded and reported.
6. Each source's credentials/config must be externalized, not hardcoded.
7. The pipeline must emit per-source stats: fetched, mapped, rejected, deduplicated.

## Suggested Milestones

1. **Milestone 1 — Connectors:** Define the connector interface and implement two sources behind it.
2. **Milestone 2 — Normalize & merge:** Map each source to the shared schema and deduplicate on the natural key.
3. **Milestone 3 — Conflicts & resilience:** Add conflict resolution, per-source error isolation, and stats.

## Data & Interface Sketch

```text
Connector (interface)
  name() -> string
  fetch() -> iterable<raw_record>
  map(raw_record) -> canonical_record | reject(reason)

canonical_record (shared schema)
  entity_id     string   (natural key, e.g. normalized email name@example.com)
  full_name     string
  email         string
  updated_at    timestamp
  _source       string
  _fetched_at   timestamp

merge:
  group by entity_id
  on conflict -> pick by precedence [db > api > csv]  OR  most-recent updated_at

source_stats: source | fetched | mapped | rejected | merged
```

## Stretch Goals

- Add a fuzzy-match step so near-duplicate names/emails collapse to one entity.
- Make each connector independently retryable with backoff on transient API errors.
- Track field-level provenance so you can answer which source supplied each value.
- Support incremental fetch per source (only records changed since last run).

## Definition of Done

- [ ] Adding a new source requires only a new connector implementation, no pipeline changes.
- [ ] Two sources describing the same entity produce one merged record, not two.
- [ ] A field conflict resolves deterministically per the documented rule.
- [ ] One source returning an error still lets the others complete, with the failure reported.
- [ ] Per-source stats reconcile: fetched = mapped + rejected.

## Common Pitfalls

- Baking source-specific logic into the core pipeline instead of behind the connector interface.
- Deduplicating on a raw field (unnormalized email/case) so real duplicates slip through.
- Resolving conflicts implicitly by last-writer-wins iteration order — make the rule explicit.
- Letting one source's timeout or auth failure crash the entire run.
- Losing which source a value came from, making later debugging impossible.

## Resources

- [Airbyte: Connector development](https://docs.airbyte.com/connector-development/) — how a production system models pluggable sources.
- [Singer specification](https://github.com/singer-io/getting-started/blob/master/docs/SPEC.md) — an open standard for source/target connectors.
- [Wikipedia: Record linkage](https://en.wikipedia.org/wiki/Record_linkage) — the theory behind matching records across sources.
- [Martin Kleppmann: Designing Data-Intensive Applications](https://dataintensive.net/) — data integration and schema evolution chapters.
