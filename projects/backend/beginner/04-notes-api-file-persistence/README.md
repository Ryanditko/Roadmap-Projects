# Notes API with File Persistence

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Beginner · **Estimated time:** 5–8 hours

## Overview

Build a notes API whose data survives a restart by writing to the file system instead of keeping everything in memory. This is the natural next step after in-memory CRUD: the same endpoints, but now every change must be reflected on disk, and the server must reload existing notes when it starts. You will confront the real questions of persistence — serialization, atomic writes, and what happens when two writes race — at the smallest possible scale.

## Prerequisites

- Experience building CRUD endpoints ([Simple REST API for Task Management](../01-simple-rest-api-task-management/) is the in-memory version of this)
- Understanding of JSON serialization and deserialization
- Familiarity with your language's file I/O and error handling
- Awareness of the working directory and relative vs absolute paths

## Learning Objectives

By the end, you should be able to:

- Read and write structured data (JSON) to the file system safely
- Load persisted state into memory on startup and keep it in sync
- Perform an atomic write so a crash mid-save cannot corrupt the store
- Handle file-system errors (missing file, permission denied) without crashing
- Assign stable unique IDs that survive restarts

## Functional Requirements

1. The system must support create, read, update, and delete for notes over HTTP.
2. Every mutation must be persisted so that data survives a server restart.
3. On startup, the system must load existing notes from disk, or start empty if none exist.
4. A note must have a unique ID that does not collide after restarts.
5. Reading, updating, or deleting a missing note must return 404.
6. The system must not corrupt the data file if a write fails or the process is killed mid-save.
7. The system must return a clear error (not a crash) when the storage file is unreadable.

## Suggested Milestones

1. **Milestone 1 — Load & list:** Read notes from a JSON file on startup and serve them; create the file if absent.
2. **Milestone 2 — Persist writes:** Make create/update/delete write changes back to disk.
3. **Milestone 3 — Safety:** Add atomic writes (write-temp-then-rename) and graceful handling of I/O errors.

## Data & Interface Sketch

```text
Storage file: notes.json
[
  { "id": "n_01", "title": "...", "body": "...", "updatedAt": "ISO-8601" }
]

GET    /notes            -> 200 [Note, ...]
GET    /notes/{id}       -> 200 Note | 404
POST   /notes            -> 201 Note   body: { title, body }
PUT    /notes/{id}       -> 200 Note | 404
DELETE /notes/{id}       -> 204 | 404

Atomic write: write notes.json.tmp -> fsync -> rename over notes.json
```

## Stretch Goals

- Add search by title and filter by an `updatedAt` date range.
- Store each note in its own file instead of one index file, and compare the trade-offs.
- Debounce writes so rapid edits batch into fewer disk operations.
- Add a timestamped backup copy before each overwrite.

## Definition of Done

- [ ] Notes created before a restart are present after it.
- [ ] The server starts cleanly whether or not the storage file already exists.
- [ ] Killing the process mid-write leaves the previous valid file intact, not a truncated one.
- [ ] Missing IDs return 404 and unreadable files return a 500 with a clear message, not a stack trace to the client.
- [ ] IDs remain unique across multiple restarts.

## Common Pitfalls

- Overwriting the file in place with a partial buffer, corrupting all notes when a write fails — use write-then-rename.
- Deriving IDs from the array length, which repeats after deletes and across restarts.
- Forgetting to create the storage file (or its directory) on first run, causing a startup crash.
- Holding the whole file open or reading it on every request instead of keeping an in-memory copy synced to disk.

## Resources

- [Node.js: fs module](https://nodejs.org/api/fs.html) — file I/O reference if you use Node.
- [Python: Reading and Writing Files](https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files) — the standard Python approach.
- [Wikipedia: Atomicity (write-rename)](https://en.wikipedia.org/wiki/Atomicity_(database_systems)) — why rename is the safe-write trick.
- [JSON.org](https://www.json.org/json-en.html) — the JSON data format at a glance.
