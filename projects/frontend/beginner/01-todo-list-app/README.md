# To-do List App

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build the classic to-do list: a single screen where a user types a task, presses Enter, sees it appear, checks it off when done, and removes it when it no longer matters. It looks trivial, but it is the smallest complete example of the core frontend loop — user input changes state, and state re-renders the UI. Everything is driven by an in-memory array of task objects that you persist to `localStorage` so the list survives a page reload. There is no server and no framework requirement; the interesting work is keeping your data model and what the user sees perfectly in sync.

## Prerequisites

- Basic HTML, CSS, and JavaScript (variables, arrays, functions)
- How to read from and write to the DOM, or a component framework of your choice
- Familiarity with array methods (`map`, `filter`, `find`)
- A code editor and a browser with dev tools

## Learning Objectives

By the end, you should be able to:

- Model UI as a function of a single source-of-truth state
- Add, toggle, edit, and delete items immutably rather than mutating in place
- Persist and rehydrate state with the `localStorage` API
- Render a filtered view (all / active / completed) without losing the underlying data
- Wire up keyboard and click interactions with accessible, labelled controls

## Functional Requirements

1. The user can type a task and add it by pressing Enter or clicking an Add button.
2. Empty or whitespace-only tasks must be rejected without adding a blank row.
3. Each task can be toggled between active and completed, with a clear visual difference.
4. The user can delete any individual task.
5. A live counter shows how many tasks remain active.
6. The user can filter the list by all, active, or completed.
7. The full list must survive a page refresh via `localStorage`.

## Suggested Milestones

1. **Milestone 1 — Add & render:** Capture input, push a task object to state, render the list.
2. **Milestone 2 — Toggle & delete:** Mark tasks done and remove them, updating the counter.
3. **Milestone 3 — Filter & persist:** Add the filter view and save/load from `localStorage`.

## Data & Interface Sketch

```text
Task
  id:        string   (crypto.randomUUID())
  title:     string
  completed: boolean
  createdAt: number   (Date.now())

Layout
+------------------------------------------+
| [ new task input............ ] [ Add ]   |
+------------------------------------------+
| [x] Buy milk                      (del)  |
| [ ] Call dentist                  (del)  |
+------------------------------------------+
| 1 item left   [All] [Active] [Completed] |
+------------------------------------------+
```

## Stretch Goals

- Inline edit: double-click a task title to rename it.
- Add a "Clear completed" button that removes all done tasks at once.
- Allow reordering tasks by drag-and-drop.
- Add a light/dark theme toggle stored alongside the tasks.

## Definition of Done

- [ ] Adding, toggling, and deleting update both the UI and stored state.
- [ ] Blank tasks cannot be added.
- [ ] The remaining-items counter is always accurate after any action.
- [ ] Filters change the visible list without discarding hidden tasks.
- [ ] Reloading the page restores the exact previous list.

## Common Pitfalls

- Mutating the state array directly instead of producing a new one, causing stale renders.
- Storing rendered HTML strings instead of a clean data model, making filters and edits painful.
- Forgetting to `JSON.parse` / `JSON.stringify` around `localStorage`, which only stores strings.
- Using array index as a key/id, which breaks after deletion and reordering.
- Skipping labels on the checkbox and buttons, leaving the app unusable with a screen reader.

## Resources

- [MDN: Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) — how `localStorage` works and its limits.
- [MDN: Introduction to the DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction) — reading and updating the page.
- [web.dev: Learn Accessibility — Forms](https://web.dev/learn/accessibility/forms) — labelling inputs and controls.
- [roadmap.sh: Frontend](https://roadmap.sh/frontend) — where these fundamentals sit in the bigger picture.
