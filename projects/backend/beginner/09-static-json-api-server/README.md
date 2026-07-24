# Static JSON API Server

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Beginner · **Estimated time:** 3–5 hours

## Overview

Build a read-only API server that loads JSON data files at startup and serves them over HTTP with filtering, pagination, and sorting — a lightweight mock backend, much like `json-server`, that a frontend can develop against before the real API exists. This is the gentlest possible introduction to backend work: no writes, no database, no auth. The whole focus is server setup, routing, and shaping query results, making it an ideal first project or a foundation the other briefs build on.

## Prerequisites

- Basic programming and the ability to run a local process
- Understanding of what JSON is
- A minimal HTTP framework or your language's built-in HTTP server
- Familiarity with URL query strings (`?key=value`)

## Learning Objectives

By the end, you should be able to:

- Stand up an HTTP server and define routes for multiple resources
- Load and cache data from JSON files at startup
- Return responses with the correct `Content-Type` and status codes
- Implement query-parameter features: filtering, sorting, and pagination
- Handle unknown routes and empty results cleanly

## Functional Requirements

1. The system must load one or more JSON data files when it starts.
2. The system must expose a listing endpoint per resource returning the data as JSON.
3. The system must expose a by-id endpoint returning a single record or 404.
4. The listing endpoint must support pagination via `limit` and `offset` (or page) parameters.
5. The listing endpoint must support filtering by at least one field via query parameter.
6. The listing endpoint must support sorting by a field, ascending and descending.
7. Unknown routes must return 404 and every response must set `Content-Type: application/json`.

## Suggested Milestones

1. **Milestone 1 — Serve data:** Load a JSON file at startup and serve the full list and a by-id lookup.
2. **Milestone 2 — Query features:** Add pagination and field filtering to the listing endpoint.
3. **Milestone 3 — Polish:** Add sorting, a health check, and consistent 404 handling.

## Data & Interface Sketch

```text
Data files loaded at boot: products.json, users.json, ...

GET /products                      -> 200 [ ... ]
GET /products/{id}                 -> 200 { ... } | 404
GET /products?category=books       -> filter by field
GET /products?_sort=price&_order=desc
GET /products?_limit=10&_offset=20 -> pagination
GET /health                        -> 200 { "status": "ok" }

Response envelope (optional):
  { "data": [ ... ], "total": 128, "limit": 10, "offset": 20 }
```

## Stretch Goals

- Add a full-text `q` search parameter across selected fields.
- Add gzip compression and cache headers (`ETag` or `Cache-Control`).
- Add CORS headers so a browser frontend can consume the API.
- Serve an auto-generated endpoint index describing the available resources.

## Definition of Done

- [ ] Data loads once at startup and is served without re-reading files per request.
- [ ] Listing supports pagination, filtering, and sorting, combinable in one request.
- [ ] A by-id lookup returns the record or a clean 404.
- [ ] Unknown routes return 404, never a crash or an HTML error page.
- [ ] Every response declares `Content-Type: application/json`.

## Common Pitfalls

- Reading and parsing the JSON file on every request instead of caching it at startup.
- Applying sort before filter (or vice versa) inconsistently, producing confusing paginated results.
- Off-by-one errors in pagination, dropping or duplicating a record at page boundaries.
- Returning an empty body for no results instead of an empty array `[]`.

## Resources

- [MDN: HTTP overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) — the fundamentals of request/response.
- [json-server](https://github.com/typicode/json-server) — a reference implementation of exactly this idea.
- [MDN: URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) — parsing query strings cleanly.
- [REST API Tutorial: filtering & pagination](https://restfulapi.net/) — conventions for query-based features.
