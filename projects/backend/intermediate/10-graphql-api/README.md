# GraphQL API with Resolvers

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build an API where the client, not the server, decides the shape of the response. Instead of a fixed set of REST endpoints, you expose a single endpoint and a typed schema; clients ask for exactly the fields they need and get exactly that — no more, no less. The interesting engineering lives underneath: resolvers that fetch each field, the notorious N+1 problem that appears the moment you resolve a list, and the batching (dataloaders) that solves it. You will also face the other side of GraphQL's flexibility — a single malicious query can ask for the world, so depth and complexity limits become a real concern.

## Prerequisites

- Comfort building an HTTP API and querying a database
- Familiarity with at least one dataset that has relationships (e.g. authors → posts → comments)
- A basic understanding of types and schemas
- A GraphQL server library for your language (Apollo, graphql-js, Strawberry, gqlgen, etc.)
- Awareness of how REST handles the same problems, for contrast

## Learning Objectives

By the end, you should be able to:

- Design a GraphQL schema with types, queries, mutations, and non-null/list modifiers
- Write resolvers that fetch data for each field, including nested relationships
- Recognize the N+1 query problem and eliminate it with a batching dataloader
- Implement cursor-based pagination over a list field
- Apply authorization inside resolvers based on the authenticated user
- Protect the server with query depth and complexity limits

## Functional Requirements

1. The API must expose a single endpoint that accepts GraphQL queries and mutations.
2. The schema must define at least two related types and resolve nested fields on demand.
3. A field the client did not request must never appear in the response.
4. Resolving a list of parents followed by a child field must not fire one query per parent (no N+1).
5. At least one list field must support cursor-based pagination returning page information.
6. Mutations must validate input and return typed errors for invalid data.
7. Resolvers must enforce authorization, hiding or rejecting fields the current user cannot access.
8. The server must reject queries that exceed a configured depth or complexity budget.

## Suggested Milestones

1. **Milestone 1 — Schema & resolvers:** Define types and write resolvers that answer nested queries against your data source.
2. **Milestone 2 — Batching & pagination:** Add a dataloader to kill the N+1, then add cursor pagination to a list.
3. **Milestone 3 — Mutations & guards:** Add validated mutations, resolver-level authorization, and depth/complexity limits.

## Data & Interface Sketch

```text
type Author {
  id: ID!
  name: String!
  posts(first: Int, after: String): PostConnection!
}
type Post { id: ID!  title: String!  author: Author! }

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
}
type PageInfo { hasNextPage: Boolean!  endCursor: String }

type Query    { author(id: ID!): Author  posts: [Post!]! }
type Mutation { createPost(input: CreatePostInput!): CreatePostPayload! }

N+1 pitfall: posts -> for each post, resolve .author => N author queries
Fix:         batch all author ids into one call via a dataloader
Guards:      reject if depth > 6 or cost > budget
```

## Stretch Goals

- Add a subscription (e.g. `postAdded`) over WebSockets for real-time updates.
- Implement field-level caching or persisted queries to cut repeated work.
- Add a custom directive (e.g. `@auth(role: ADMIN)`) enforced during execution.
- Expose query cost in a response extension so clients can self-tune.

## Definition of Done

- [ ] A client can request a subset of fields and receives precisely those fields, deeply nested.
- [ ] Resolving a list plus a related field fires a bounded number of queries, verified by logs.
- [ ] A paginated list returns stable cursors and correct `hasNextPage` across pages.
- [ ] Invalid mutation input returns a typed, actionable error instead of a 500.
- [ ] A too-deep or too-complex query is rejected before execution, not after.

## Common Pitfalls

- Ignoring the N+1 problem until a list query fans out into hundreds of database calls.
- Putting business/authorization logic in the transport layer instead of resolvers, so it gets bypassed.
- Returning `null` for a non-null field (`!`), which nullifies the entire parent object unexpectedly.
- Offset pagination over mutable data, causing skipped or duplicated items between pages.
- No complexity limit, letting a deeply nested query become an accidental denial of service.

## Resources

- [GraphQL: Official Learn](https://graphql.org/learn/) — schema, queries, resolvers, and execution from the source.
- [Apollo: Understanding the N+1 problem](https://www.apollographql.com/docs/technotes/TN0019-avoiding-the-n-plus-1/) — why it happens and how dataloaders fix it.
- [GraphQL Cursor Connections Spec](https://relay.dev/graphql/connections.htm) — the standard shape for cursor pagination.
- [OWASP: GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html) — depth/complexity limits and other guards.
