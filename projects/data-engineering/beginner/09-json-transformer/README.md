# JSON Transformer

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Two systems rarely agree on the shape of their JSON. One nests an address deep inside a customer object; the other wants a flat row with `customer_city`. Build a tool that reshapes JSON from a source structure to a target structure using a declarative mapping — flattening nested objects, renaming keys, picking fields out of arrays, and casting types. The goal is a small transformation *engine* driven by configuration rather than one-off code, so the same tool can adapt document A into document B without you rewriting logic each time the schema changes.

## Prerequisites

- Solid understanding of JSON (objects, arrays, nesting, null)
- Comfort navigating nested data structures in code
- Familiarity with recursion or iterative traversal of trees
- Beginner error handling for missing or unexpected fields

## Learning Objectives

By the end, you should be able to:

- Traverse arbitrarily nested JSON safely, handling missing paths
- Express a transformation as a source-path → target-path mapping
- Flatten nested objects into dotted or underscored keys and back (nest)
- Handle arrays: pick an element, map over all, or aggregate (count, join)
- Cast values to a target type and default or reject on mismatch
- Validate output against an expected shape before emitting it

## Functional Requirements

1. The tool must read JSON input (a single document or newline-delimited records) and a mapping spec.
2. It must resolve nested source paths (e.g. `address.city`) and place values at target paths, missing paths handled explicitly.
3. It must support flattening (nested → flat) and nesting (flat → nested) driven by the mapping.
4. It must handle arrays with a declared rule: index, map-over, or aggregate.
5. Values must be cast to declared target types; a failed cast must default or route the record to rejects per config.
6. Transformed output must be validated against a target shape and written; a summary reports transformed and rejected counts.

## Suggested Milestones

1. **Milestone 1 — Rename & pick:** Map top-level fields and copy selected keys to the target.
2. **Milestone 2 — Nested & arrays:** Resolve nested paths, flatten/nest, and apply an array rule.
3. **Milestone 3 — Types & validation:** Add type casting, defaults/rejects, output validation, and a summary.

## Data & Interface Sketch

```text
source document
  { "id": 7,
    "customer": { "name": "Ana", "address": { "city": "Recife" } },
    "items": [ {"sku":"A","qty":2}, {"sku":"B","qty":1} ] }

mapping spec
  id                       -> order_id        (cast: int)
  customer.name            -> customer_name
  customer.address.city    -> customer_city   (default: "unknown")
  items[*].qty             -> total_qty        (aggregate: sum)
  items[0].sku             -> first_sku

target document (flat)
  { "order_id": 7, "customer_name": "Ana",
    "customer_city": "Recife", "total_qty": 3, "first_sku": "A" }

on missing path -> use default OR reject (per field config)
summary: transformed=980 rejected=20
```

## Stretch Goals

- Support a small expression syntax for derived fields (concatenation, arithmetic).
- Add reverse mapping so a transform can be inverted (target → source).
- Validate against a JSON Schema instead of an ad-hoc shape.
- Stream newline-delimited JSON so huge files transform without buffering.

## Definition of Done

- [ ] Nested source paths resolve correctly, with missing paths handled per config.
- [ ] Flatten and nest both work, driven only by the mapping spec.
- [ ] Array rules (index, map, aggregate) produce correct target values.
- [ ] Type casts succeed or trigger the configured default/reject behavior.
- [ ] Output is validated against the target shape and a summary is reported.

## Common Pitfalls

- Crashing on a missing nested key instead of applying a default or rejecting cleanly.
- Hard-coding the transformation so a schema change means editing code, not config.
- Losing array data by grabbing `[0]` when you meant to aggregate over all elements.
- Emitting invalid output because you skip validating the transformed shape.

## Resources

- [JSON specification (ECMA-404)](https://www.json.org/json-en.html) — the precise data model you are traversing.
- [JMESPath](https://jmespath.org/) — a query language for extracting and reshaping JSON, great inspiration for path syntax.
- [jq manual](https://jqlang.github.io/jq/manual/) — the canonical JSON transformation tool to compare against.
- [JSON Schema validation](https://json-schema.org/understanding-json-schema/) — how to validate your target output rigorously.
