# Schema Evolution System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a system that lets a data schema change over time without breaking the producers and consumers that depend on it. Real pipelines never freeze their schema: fields get added, renamed, deprecated, and occasionally removed — and yet last year's data must still be readable and last week's consumer must not crash on today's records. You will implement a schema registry that versions schemas, classifies each proposed change as backward-, forward-, or fully-compatible, and rejects breaking changes before they ship. You will then write migrations that upgrade old records to the current schema on read. The lesson is that schema is a contract, and evolving a contract safely is a discipline, not an afterthought.

## Prerequisites

- Comfort modeling data with a serialization format (JSON Schema, Avro, or Protobuf)
- Understanding of producers and consumers sharing a data contract
- Familiarity with semantic versioning concepts
- The [Kafka streaming project](../03-streaming-kafka/) is useful context for why compatibility matters

## Learning Objectives

By the end, you should be able to:

- Version schemas and store them in a registry keyed by subject and version
- Distinguish backward, forward, and full compatibility and reason about who breaks when
- Detect a breaking change (e.g. removing a required field) before it is registered
- Migrate historical records to the current schema on read
- Manage field deprecation with defaults instead of hard removal

## Functional Requirements

1. The registry must store multiple versions of a schema under a stable subject name.
2. Registering a new version must run a compatibility check against the previous version(s).
3. A backward-incompatible change (e.g. dropping/renaming a required field) must be rejected with a clear reason.
4. Adding an optional field with a default must be accepted as a backward-compatible change.
5. A reader must be able to deserialize a record written with any registered version into the latest schema.
6. Deprecated fields must be supported via defaults so old and new records both read cleanly.
7. Every registered version must be retrievable by number so historical data stays decodable.

## Suggested Milestones

1. **Milestone 1 — Versioned registry:** Store and retrieve schemas by subject and version number.
2. **Milestone 2 — Compatibility checks:** Classify a proposed change and reject breaking ones automatically.
3. **Milestone 3 — Read-time migration:** Upgrade records from any old version to the latest on read.

## Data & Interface Sketch

```text
registry:
  subject "user"
    v1: { id: int, name: string }
    v2: { id: int, name: string, email: string = "" }   # +optional, default -> backward-compatible
    v3: { id: int, full_name: string }                    # rename required -> REJECTED

compatibility(new, old) -> BACKWARD | FORWARD | FULL | NONE
  add optional w/ default        -> BACKWARD ok
  remove/rename required field   -> NONE     -> reject
  add required field w/o default -> NONE     -> reject

read(record):
  detect writer_version from record
  apply migrations v_writer -> v_latest (fill defaults, map renames)
  return record shaped as v_latest
```

## Stretch Goals

- Support a compatibility *mode* per subject (backward / forward / full) and enforce accordingly.
- Add a dry-run "what would break" report for a proposed schema before registering it.
- Handle a controlled rename via an alias that reads both the old and new field name.
- Integrate with Avro or Protobuf and reuse its native resolution rules.

## Definition of Done

- [ ] Schemas are stored and retrievable by subject and version, with an immutable history.
- [ ] Adding an optional defaulted field is accepted; removing a required field is rejected with a reason.
- [ ] A record written under v1 deserializes correctly against the latest schema.
- [ ] Deprecated fields resolve via defaults with no reader errors.
- [ ] The compatibility classifier is covered by tests for each change type.

## Common Pitfalls

- Treating "add a field" as always safe — a new *required* field without a default breaks old producers.
- Mutating a registered version in place instead of creating a new one, corrupting history.
- Confusing backward and forward compatibility and enforcing the wrong direction for your rollout order.
- Renaming a field and calling it compatible; to readers it is a drop plus an add.
- Relying on field order or position instead of names, so any reorder silently misreads data.

## Resources

- [Confluent: Schema evolution and compatibility](https://docs.confluent.io/platform/current/schema-registry/fundamentals/schema-evolution.html) — the definitive compatibility taxonomy.
- [Apache Avro: Schema Resolution](https://avro.apache.org/docs/current/specification/#schema-resolution) — how readers and writers reconcile schemas.
- [JSON Schema documentation](https://json-schema.org/understanding-json-schema/) — modeling and validating evolving schemas.
- [Protocol Buffers: Updating a message type](https://protobuf.dev/programming-guides/proto3/#updating) — field-number rules for safe evolution.
