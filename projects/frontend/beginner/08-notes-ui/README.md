# Notes UI (Local Storage)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build a note-taking app: a list of notes on one side, an editor on the other, and everything saved to `localStorage` so it survives a reload. It is the To-do list grown up — instead of one-line items you have titled, editable documents, and instead of a single action you have full create-read-update-delete. The core challenge is the master-detail pattern: a list and an editor that stay in sync, where selecting a note loads it, editing updates it live, and deleting cleans up without leaving a dangling selection. Add search and auto-save and it starts to feel like a real tool.

## Prerequisites

- HTML, CSS, and JavaScript fundamentals
- The `localStorage` API and `JSON` serialization
- Array/object manipulation for CRUD operations
- Debouncing basics for auto-save (or a timer)

## Learning Objectives

By the end, you should be able to:

- Implement full CRUD over a collection persisted in `localStorage`
- Build a master-detail layout where a list and editor stay synchronized
- Auto-save edits with debouncing instead of a manual save button
- Search/filter notes by title or content in real time
- Handle edge cases: no notes, deleting the selected note, empty titles

## Functional Requirements

1. The user can create a new note, which appears in the list and opens in the editor.
2. Editing a note's title or body updates the list and persists automatically.
3. The user can delete a note, with a confirmation, and the selection resets sensibly.
4. A search box filters the note list by title or content.
5. Notes persist across reloads via `localStorage`.
6. Each note shows a last-updated timestamp that reflects real edits.
7. An empty state is shown when there are no notes or no search matches.

## Suggested Milestones

1. **Milestone 1 — List & create:** Render the note list and support creating and selecting notes.
2. **Milestone 2 — Edit & persist:** Edit title/body, auto-save to `localStorage`, update timestamps.
3. **Milestone 3 — Delete & search:** Add delete with confirmation and live search over notes.

## Data & Interface Sketch

```text
Note
  id:        string   (crypto.randomUUID())
  title:     string
  body:      string
  updatedAt: number   (Date.now())

App state
  notes:      Note[]
  selectedId: string | null
  query:      string

Layout (master-detail)
+------------------+-----------------------------+
| [ search....... ]| Title [ ................. ] |
| + New note       |                             |
|------------------|  body textarea              |
| > Groceries   ·  |  ...                        |
|   Meeting     ·  |                             |
|   Ideas       ·  |  updated 2m ago    [Delete] |
+------------------+-----------------------------+
```

## Stretch Goals

- Add pinning so important notes stay at the top of the list.
- Add tags or folders and filter by them.
- Support basic Markdown rendering in a preview pane.
- Add export/import of all notes as a single JSON file.

## Definition of Done

- [ ] Create, edit, and delete all update the list, editor, and stored data.
- [ ] Edits auto-save without a manual save action.
- [ ] Deleting the selected note leaves a sensible selection or empty state.
- [ ] Search filters the list in real time and shows an empty state when nothing matches.
- [ ] All notes survive a page reload intact.

## Common Pitfalls

- Writing to `localStorage` on every keystroke without debouncing, causing jank.
- Losing the current selection or crashing when the selected note is deleted.
- Keeping the editor's value out of sync with the stored note after a switch.
- Exceeding the `localStorage` quota with large notes and not handling the error.
- Forgetting the empty state, so a fresh app looks broken.

## Resources

- [MDN: Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) — persisting and reading structured data.
- [MDN: JSON.stringify()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) — serializing your note collection.
- [CSS-Tricks: Debouncing and Throttling](https://css-tricks.com/debouncing-throttling-explained-examples/) — rate-limiting auto-save.
- [web.dev: Learn Forms — textarea](https://web.dev/learn/forms/textareas) — accessible multi-line input.
