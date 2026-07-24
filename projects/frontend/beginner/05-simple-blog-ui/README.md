# Simple Blog UI

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build the reading side of a blog: a list of posts you can browse, search, and filter by category, plus a detail view for reading a single post in full. There is no writing or backend — posts come from a local JSON file or a small in-memory array — so the focus stays on presenting content clearly and moving between a list and a detail without a full page reload. This is where beginners first meet the idea of a "view": the same data rendered two ways, and simple client-side navigation that keeps the URL and the screen in agreement.

## Prerequisites

- HTML, CSS, and JavaScript fundamentals
- Array methods (`filter`, `map`, `find`) for list transformations
- Basic understanding of client-side routing or show/hide view switching
- A component framework of your choice is optional but welcome

## Learning Objectives

By the end, you should be able to:

- Render a list of items from a data source and a detail view for one item
- Implement client-side search and category filtering over the same dataset
- Navigate between list and detail views while keeping state consistent
- Paginate or lazily reveal a long list without overwhelming the DOM
- Present readable typography and an accessible reading order

## Functional Requirements

1. The home view lists posts with title, excerpt, category, and estimated reading time.
2. Selecting a post opens a detail view showing its full content.
3. A search box filters the list by matching title or excerpt text.
4. Category filters narrow the list; clearing them restores the full set.
5. A long list is paginated or uses "load more" rather than rendering everything at once.
6. The user can return from a detail view to the list without losing their filter.
7. Each post detail has a single `h1` and a logical heading structure.

## Suggested Milestones

1. **Milestone 1 — List & detail:** Load posts and render the list, then a full detail view on selection.
2. **Milestone 2 — Search & filter:** Add text search and category filtering over the dataset.
3. **Milestone 3 — Navigation & paging:** Preserve filters across views and paginate the list.

## Data & Interface Sketch

```text
Post
  id:          string
  title:       string
  excerpt:     string
  body:        string
  category:    string
  publishedAt: string   (ISO-8601)
  readMinutes: number

Views
  list   -> filtered/paged array of Post summaries
  detail -> one Post by id

Layout (list)                Layout (detail)
+-------------------------+   +----------------------+
| [ search ] [Category v] |   | < Back               |
+-------------------------+   | Title (h1)           |
| Post card               |   | meta: cat · 5 min    |
| Post card               |   |                      |
| Post card               |   | body paragraphs...   |
+-------------------------+   +----------------------+
| < 1 2 3 >               |
+-------------------------+
```

## Stretch Goals

- Sync the current view and filters to the URL (query params or hash) so links are shareable.
- Add a tag cloud and cross-link related posts by shared tags.
- Add a table of contents generated from the post's headings.
- Show a "no results" empty state with a way to reset filters.

## Definition of Done

- [ ] The list and detail views render the same data correctly from one source.
- [ ] Search and category filters combine and can be cleared.
- [ ] Returning to the list preserves the active search and filter.
- [ ] A long list does not render all posts to the DOM at once.
- [ ] Each detail page has exactly one `h1` and ordered headings.

## Common Pitfalls

- Duplicating post data for the list and detail instead of deriving both from one source.
- Losing the active filter when navigating back from a detail view.
- Case-sensitive search that misses obvious matches.
- Rendering hundreds of cards up front, making the page janky.
- Forgetting an empty state, so a filtered-out list looks like a bug.

## Resources

- [MDN: Array.prototype.filter()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) — building search and category filtering.
- [MDN: History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API) — reflecting views in the URL.
- [web.dev: Learn Accessibility — Content structure](https://web.dev/learn/accessibility/structure) — headings and reading order.
- [MDN: Working with JSON](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON) — loading and parsing local data.
