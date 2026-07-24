# Blog Platform with CRUD and Comments

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build the API behind a blogging platform: authors write posts, readers leave comments, and everything is organized by tags and a draft/publish workflow. What lifts this above a plain CRUD exercise is the relationships — comments that thread into replies, posts owned by their authors, and moderation that hides content without destroying it. You will wrestle with nested resources, pagination over potentially large lists, and permission checks that answer "can *this* user edit *that* post?" cleanly on every request.

## Prerequisites

- Comfort building REST resources ([Simple REST API for Task Management](../../beginner/01-simple-rest-api-task-management/) is a good warm-up)
- Basic authentication and sessions or tokens ([Basic Authentication API](../../beginner/03-basic-authentication-api/))
- A database with support for relations or references (PostgreSQL, MySQL, or MongoDB)
- Understanding of one-to-many relationships and foreign keys

## Learning Objectives

By the end, you should be able to:

- Model nested resources (posts → comments → replies) in a REST API
- Implement pagination and reason about offset vs. cursor trade-offs
- Enforce ownership and role-based permissions on write operations
- Implement soft deletes so content can be hidden and later recovered
- Filter and search posts by tag efficiently

## Functional Requirements

1. Authors must be able to create, update, and delete their own posts; others must be forbidden with 403.
2. A post must have a status of `draft` or `published`; drafts must not appear in public listings.
3. Any authenticated user must be able to comment on a published post and reply to existing comments.
4. Comment listings must return replies nested (or resolvable) under their parent.
5. Post listings must be paginated and must support filtering by one or more tags.
6. Deleting a post or comment must be a soft delete — the record is hidden, not physically removed.
7. A moderator/admin must be able to hide (reject) any comment regardless of authorship.

## Suggested Milestones

1. **Milestone 1 — Posts CRUD:** Create, read, update, delete posts with ownership checks and the draft/publish status.
2. **Milestone 2 — Comments & threading:** Add comments and nested replies, returning them in a readable tree.
3. **Milestone 3 — Discovery:** Tag filtering, search, and paginated listings for posts and comments.
4. **Milestone 4 — Moderation & soft delete:** Hide/recover posts and comments and expose a moderation endpoint.

## Data & Interface Sketch

```text
Post     id, authorId, title, body, tags[], status, createdAt, deletedAt?
Comment  id, postId, authorId, parentId?, body, hidden, createdAt, deletedAt?

POST /posts                 (author)  -> 201 post
GET  /posts?tag=go&page=2             -> 200 { items, page, total }
PATCH /posts/{id}           (owner)   -> 200 post | 403
POST /posts/{id}/comments             -> 201 comment
GET  /posts/{id}/comments             -> 200 [ { ...comment, replies: [...] } ]
DELETE /comments/{id}       (soft)    -> 204
POST /comments/{id}/hide    (mod)     -> 200

Pagination: ?page=&limit=  (or cursor: ?after=<id>)
```

## Stretch Goals

- Add a follow feature so a user sees a feed of posts from authors they follow.
- Track view counts and expose a "most read this week" listing.
- Support Markdown in post bodies, rendering to sanitized HTML.
- Add related-posts suggestions based on shared tags.

## Definition of Done

- [ ] A user cannot edit or delete a post or comment they do not own (unless moderator).
- [ ] Draft posts never appear in any public/anonymous listing.
- [ ] Replies are correctly associated with their parent comment and returned nested.
- [ ] Soft-deleted content disappears from reads but remains recoverable in storage.
- [ ] Listings are paginated and tag filtering returns only matching posts.

## Common Pitfalls

- Loading all comments and building the reply tree in memory for every request — paginate and bound depth.
- Forgetting to exclude soft-deleted rows from every query, so "deleted" content reappears.
- Checking ownership after performing the write instead of before it.
- Leaking draft posts through a search or tag endpoint that skips the status filter.
- Offset pagination that shifts results when new posts are inserted mid-scroll.

## Resources

- [MDN: HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) — pick the right code for forbidden, not-found, and no-content.
- [PostgreSQL: Soft delete patterns](https://www.postgresql.org/docs/current/ddl-partitioning.html) — background on filtering and organizing rows.
- [Use The Index, Luke: Pagination](https://use-the-index-luke.com/no-offset) — why cursor pagination beats offset at scale.
- [roadmap.sh: Backend Developer](https://roadmap.sh/backend) — where data modeling and APIs fit overall.
