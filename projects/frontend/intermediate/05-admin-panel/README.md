# Admin Panel (CRUD)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build the back-office screen every product eventually needs: a table of records that an operator can read, create, edit, and delete against a real API. The interesting part is not the buttons — it is making a data table stay honest while the server is the source of truth. You will juggle server-side sorting, filtering, and pagination, keep forms and validation in sync with the table, guard destructive actions behind confirmation, and reflect a user's role in what they are allowed to touch. It is a compact tour of the state, data-fetching, and form concerns that dominate real internal tooling.

## Prerequisites

- Comfort building components and managing local state in a component framework (React, Vue, Svelte, or Angular)
- Fetching data from a REST API and handling promises/async
- Basic understanding of controlled form inputs and client-side validation
- Familiarity with query strings and how they carry state (`?page=2&sort=name`)

## Learning Objectives

By the end, you should be able to:

- Drive a table from server state (sort, filter, paginate) instead of loading everything at once
- Model loading, empty, error, and success as explicit UI states rather than an afterthought
- Build create/edit forms that validate before submitting and surface field-level errors
- Protect destructive operations with confirmation and optimistic-or-pending feedback
- Reflect a user role in the interface, disabling or hiding actions the role cannot perform
- Keep table state shareable and restorable via the URL

## Functional Requirements

1. The table must list records with columns, and support sorting by at least two columns via the server.
2. Filtering (by a text query and at least one field facet) must re-query the server, not filter a stale local array.
3. Pagination must request one page at a time and show total count / current range.
4. A "New" action must open a form that validates required fields and formats before allowing submit.
5. Editing an existing record must pre-fill the form and persist changes via the API.
6. Deleting must require an explicit confirmation step and must not fire on a single stray click.
7. Every async action must render a loading state and a recoverable error state (with retry).
8. Actions the current role lacks permission for must be disabled or hidden, with an accessible label explaining why.

## Suggested Milestones

1. **Milestone 1 — Read:** Fetch and render the table with loading/empty/error states.
2. **Milestone 2 — Query:** Add server-driven sorting, filtering, and pagination, synced to the URL.
3. **Milestone 3 — Write:** Add create and edit forms with validation and API persistence.
4. **Milestone 4 — Delete & roles:** Add confirmed deletion, bulk selection, and role-gated actions.

## Data & Interface Sketch

```text
Layout
+---------------------------------------------------------+
| Toolbar: [search] [role: admin ▾]        [+ New record] |
+---------------------------------------------------------+
| [ ] Name ▲    | Email          | Role   | Status |  ... |
| [x] Ada L.    | ada@example.com     | admin  | active |  ⋯   |
| ...                                                      |
+---------------------------------------------------------+
| Selected: 2   [Delete]      Page 2 of 9  ‹ 21–40 / 174 ›|
+---------------------------------------------------------+

State shape
  records: Record[]        query: { q, sort, dir, page, pageSize, filters }
  status: 'idle'|'loading'|'error'|'ready'
  selection: Set<id>       editing: Record | null

API contract consumed
  GET  /api/records?q=&sort=name&dir=asc&page=2&pageSize=20
       -> 200 { items: Record[], total: 174 }
  POST /api/records        body: {..}  -> 201 Record | 422 { errors }
  PUT  /api/records/{id}   body: {..}  -> 200 Record | 422 { errors }
  DELETE /api/records/{id}             -> 204 | 403
```

## Stretch Goals

- Add bulk operations (delete/deactivate many) with a single confirmation summarising the count.
- Persist column visibility and page size to local storage per user.
- Add optimistic updates for edits, rolling back on a failed request.
- Export the current filtered view to CSV.

## Definition of Done

- [ ] Sorting, filtering, and pagination all round-trip to the server and survive a page refresh via the URL.
- [ ] Create and edit forms block submission on invalid input and show field-level errors from a 422.
- [ ] Deletion always passes through an explicit confirmation and never fires on a single click.
- [ ] Loading, empty, and error states are visible and the error state offers retry.
- [ ] Role-restricted actions are disabled or hidden with an accessible explanation.

## Common Pitfalls

- Fetching the whole dataset and sorting/filtering on the client — it works with 20 rows and dies with 20,000.
- Treating loading and error as the absence of data, leaving the user staring at an empty table.
- Losing table state on refresh because it lives only in memory, not the URL.
- Firing a delete on row click without confirmation, then having no undo.
- Trusting the client-side role check for security — it is a UX affordance; the server must still enforce permissions.

## Resources

- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) — making and handling API requests.
- [MDN: Client-side form validation](https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation) — validating input the right way.
- [web.dev: Accessible data tables](https://web.dev/articles/grid-role) — semantics and keyboard behaviour for grids.
- [ARIA Authoring Practices: Grid pattern](https://www.w3.org/WAI/ARIA/apg/patterns/grid/) — the interaction contract for editable tables.
- [roadmap.sh: Frontend](https://roadmap.sh/frontend) — where these skills sit in the bigger picture.
