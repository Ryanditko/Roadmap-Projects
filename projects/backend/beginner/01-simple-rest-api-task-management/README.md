# Simple REST API for Task Management

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Beginner · **Estimated time:** 4–7 hours

## Overview

Build a REST API that manages a to-do list entirely in memory, with no database involved. Imagine the backend behind a simple task app: the server keeps tasks in a list while it runs, and clients create, read, update, and delete them over HTTP. This is the classic first backend project because it isolates the core skill — mapping HTTP verbs to operations on a resource — without the distraction of persistence.

## Prerequisites

- Basic understanding of HTTP: what a request, response, method, and status code are
- Comfort with your language's package manager and running a local process
- A web framework of your choice (Express for Node, Flask/FastAPI for Python, Gin for Go)
- A tool to send requests: `curl`, HTTPie, Postman, or an editor REST client

## Learning Objectives

By the end, you should be able to:

- Map the CRUD operations onto the correct HTTP methods (GET, POST, PUT/PATCH, DELETE)
- Choose meaningful status codes (200, 201, 204, 400, 404) for each outcome
- Parse a JSON request body and serialize a JSON response
- Validate incoming data before mutating your in-memory store
- Generate stable unique identifiers for new resources
- Explain why in-memory state disappears on restart and when that is acceptable

## Functional Requirements

1. The system must expose an endpoint to list all tasks.
2. The system must expose an endpoint to fetch a single task by its ID and return 404 when it does not exist.
3. The system must create a task from a JSON body, assign it a unique ID, and return 201 with the created resource.
4. The system must reject creation requests missing a required field with a 400 and a helpful error message.
5. The system must update an existing task and return 404 if the ID is unknown.
6. The system must delete a task by ID and return 204 (or 404 if absent).
7. The system must return valid JSON with the correct `Content-Type` header on every response.

## Suggested Milestones

1. **Milestone 1 — Read path:** Serve a hardcoded array of tasks through GET (list) and GET by ID, including the 404 case.
2. **Milestone 2 — Write path:** Add POST with ID generation and DELETE, mutating the in-memory list.
3. **Milestone 3 — Robustness:** Add PUT/PATCH, input validation with 400 responses, and consistent error shapes.

## Data & Interface Sketch

```text
Task
  id:        string | number   (server-assigned)
  title:     string            (required)
  completed: boolean           (default false)
  createdAt: ISO-8601 string

GET    /tasks            -> 200 [Task, ...]
GET    /tasks/{id}       -> 200 Task | 404
POST   /tasks            -> 201 Task    body: { title }
PUT    /tasks/{id}       -> 200 Task | 404
DELETE /tasks/{id}       -> 204 | 404

Error shape: { "error": "title is required" }
```

## Stretch Goals

- Add a `?completed=true` query filter to the list endpoint.
- Support PATCH for partial updates alongside full PUT.
- Add pagination with `limit` and `offset` query parameters.
- Write automated tests that exercise each endpoint and status code.

## Definition of Done

- [ ] All five CRUD endpoints work and return the documented status codes.
- [ ] Fetching, updating, or deleting a missing ID returns 404, not a crash.
- [ ] Invalid input returns 400 with a clear message, never a 500.
- [ ] IDs are unique even when tasks are created in quick succession.
- [ ] Every response sets `Content-Type: application/json`.

## Common Pitfalls

- Returning 200 for a created resource instead of 201, or 200 for a missing one instead of 404 — reviewers spot this immediately.
- Using array index as the ID, which breaks after a delete shifts the list.
- Forgetting to parse the JSON body, leaving your handler with `undefined`/`None` fields.
- Letting an unhandled exception leak a 500 when a clean 400/404 was the correct answer.

## Resources

- [MDN: HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) — the canonical reference for what each verb means.
- [MDN: HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) — pick the right code for every outcome.
- [REST API Tutorial](https://restfulapi.net/) — practical conventions for designing resource endpoints.
- [roadmap.sh: Backend Developer](https://roadmap.sh/backend) — where this skill sits in the bigger picture.
