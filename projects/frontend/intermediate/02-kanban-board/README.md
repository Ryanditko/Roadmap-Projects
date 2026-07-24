# Kanban Board (Drag and Drop)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a Kanban board — columns like *To Do*, *In Progress*, and *Done* holding draggable cards — that a person can rearrange by grabbing a card and dropping it into another column. The satisfying part is the drag itself, but the real learning is underneath it: modelling the board as normalized state, moving a card between two lists without corrupting order, persisting the layout so it survives a reload, and making all of it operable without a mouse. Drag-and-drop is notoriously inaccessible when done naively, so a keyboard-and-screen-reader path is a first-class requirement here, not an afterthought.

## Prerequisites

- Comfort building components and managing state in a framework (React, Vue, Svelte, or Angular)
- Understanding of arrays and immutable updates (moving an item between two lists)
- Familiarity with the browser event model (pointer, drag, or keyboard events)
- Basic use of `localStorage` for persistence
- Optional: awareness of a drag-and-drop library (dnd kit, SortableJS) versus the native HTML Drag and Drop API

## Learning Objectives

By the end, you should be able to:

- Model a board as normalized state (columns, cards, and an explicit order) rather than nested arrays
- Implement moving a card within and across columns without losing or duplicating it
- Choose between the native Drag and Drop API and a library, and justify the trade-off
- Provide a keyboard-operable alternative to dragging (grab, move, drop)
- Announce reorder and move actions to assistive technology
- Persist and rehydrate board state across page reloads

## Functional Requirements

1. The board must render multiple columns, each with an ordered list of cards.
2. A card must be draggable from one column and droppable into any column at a chosen position.
3. Reordering must update the underlying order deterministically — no duplicate or lost cards.
4. Users must be able to create, edit, and delete cards, and edits must open a card detail form.
5. The full board state must persist to `localStorage` and restore on reload.
6. Every drag interaction must have a keyboard equivalent (pick up, move between columns, drop).
7. Drag targets must show a clear drop indicator, and moves must be announced via an ARIA live region.

## Suggested Milestones

1. **Milestone 1 — Static board:** Render columns and cards from normalized state; add card create/edit/delete.
2. **Milestone 2 — Drag to move:** Implement pointer drag-and-drop within and across columns with a drop indicator.
3. **Milestone 3 — Persist:** Save and rehydrate board state from `localStorage`.
4. **Milestone 4 — Keyboard & a11y:** Add a keyboard move path and live-region announcements.

## Data & Interface Sketch

```text
Layout
  ┌──────────┬──────────────┬──────────┐
  │  To Do   │ In Progress  │   Done   │
  ├──────────┼──────────────┼──────────┤
  │ [card 1] │ [card 4]     │ [card 6] │
  │ [card 2] │ [card 5]     │          │
  │ [card 3] │  ▁▁drop▁▁    │          │  ← drop indicator
  │ [+ add]  │ [+ add]      │ [+ add]  │
  └──────────┴──────────────┴──────────┘

State (normalized)
  columns:     { id, title, cardIds: string[] }[]   // order lives here
  cards:       Record<id, { id, title, description, priority }>
  dragging:    { cardId, fromColumn } | null

Card  { id, title, description, priority: 'low'|'med'|'high' }

Move operation (pure)
  move(state, cardId, toColumn, toIndex) -> new state
  // remove id from source cardIds, insert into target at toIndex
```

## Stretch Goals

- Add card filtering or a search box that dims non-matching cards.
- Support multiple boards, switchable from a sidebar.
- Add undo/redo for move and delete actions.
- Add a "priority" swimlane view that groups cards regardless of column.
- Sync board state to a backend API instead of `localStorage`.

## Definition of Done

- [ ] Cards move within and between columns with order preserved and no duplicates.
- [ ] The board survives a full reload with columns and card order intact.
- [ ] Every move achievable by drag is also achievable by keyboard alone.
- [ ] A drop indicator shows where a card will land before release.
- [ ] Move and reorder actions are announced to screen readers.

## Common Pitfalls

- Storing cards nested inside columns, making cross-column moves an error-prone deep copy — normalize instead.
- Relying only on native drag events, which are unusable by keyboard and flaky on touch screens.
- Mutating the order array in place, causing React/Vue to miss the change or render stale order.
- Forgetting stable keys, so cards visually jump or lose focus during a reorder.
- Treating accessibility as a stretch goal — retrofitting a keyboard path onto pointer-only drag is painful.

## Resources

- [MDN: HTML Drag and Drop API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API) — the native primitives and their limits.
- [dnd kit documentation](https://docs.dndkit.com/) — an accessible, keyboard-friendly drag-and-drop toolkit.
- [W3C ARIA APG: Live regions](https://www.w3.org/WAI/ARIA/apg/practices/live-regions/) — announcing dynamic moves.
- [web.dev: Keyboard access](https://web.dev/articles/keyboard-access) — making interactions operable without a mouse.
- [MDN: Window.localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) — persisting the board.
