# Visual Page Builder (drag/drop)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build a drag-and-drop editor where non-developers compose pages by dropping components onto a canvas, nesting them, editing their properties, and previewing the result live — the pattern behind Webflow, Framer, and Notion's block editor. The deceptively simple drag interaction hides the real challenge: a document model. Every page is a serializable tree of nodes, and every action (drop, move, edit, delete) is a well-defined transformation of that tree that must round-trip perfectly to storage, support undo/redo, and render both an editable canvas and a clean output. This project is fundamentally about modelling a tree and building predictable operations over it, with drag-and-drop as the visible surface.

## Prerequisites

- Strong command of component composition and state management
- Comfort modelling recursive/tree data structures
- Understanding of the HTML Drag and Drop API or a pointer-based dragging approach
- Familiarity with immutable update patterns for predictable state

## Learning Objectives

By the end, you should be able to:

- Model a page as a serializable component tree with a stable node schema
- Implement drag-and-drop that inserts and reorders nodes, including nesting
- Build a property editor that edits the selected node and reflects changes live
- Implement undo/redo as inverse operations over the document tree
- Serialize to and deserialize from storage so a saved page reloads identically

## Functional Requirements

1. Users must drag components from a palette onto a canvas and see them render immediately.
2. Components must be nestable (a container can hold children) with clear valid/invalid drop targets.
3. Selecting a component must open a property editor whose changes update the canvas live.
4. Every mutating action must be undoable and redoable in correct order.
5. The page must serialize to a portable format and deserialize back to an identical tree.
6. The builder must render a clean preview/output that matches the canvas without editor chrome.
7. Reordering and moving nodes must keep the tree valid (no cycles, no orphaned nodes).

## Suggested Milestones

1. **Milestone 1 — Tree & render:** Define the node schema and render a static tree to a canvas.
2. **Milestone 2 — Drag, drop, nest:** Implement dragging from the palette and reordering/nesting on the canvas.
3. **Milestone 3 — Properties & preview:** Add the property editor and a chrome-free preview mode.
4. **Milestone 4 — History & persistence:** Add undo/redo and serialize/deserialize to storage.

## Data & Interface Sketch

```text
   Document tree (serializable):
   {
     id, type: "Page",
     children: [
       { id, type: "Container", props: {...}, children: [
           { id, type: "Text",  props: { content } },
           { id, type: "Image", props: { src, alt } }
       ]}
     ]
   }

   Palette ──drag──▶ Canvas (renders tree) ──select──▶ Property editor
                         │                                   │
                         └──── mutations (insert/move/edit/delete) ──┐
                                                                     ▼
                                              History: [op, inverse-op] for undo/redo

Operations (pure, tree -> tree):
  insert(parentId, index, node)   move(nodeId, newParentId, index)
  updateProps(nodeId, patch)      remove(nodeId)
Invariants:  no cycles · every node reachable · ids unique
```

## Stretch Goals

- Add responsive preview at multiple breakpoints with per-breakpoint prop overrides.
- Add snap-to-grid and alignment guides for precise placement.
- Support reusable component templates/symbols that update all instances.
- Add optional multi-user editing on top of the tree model.

## Definition of Done

- [ ] A component dragged from the palette renders on the canvas and can be nested in a container.
- [ ] Editing a property in the panel updates the canvas immediately.
- [ ] Undo and redo correctly reverse and replay every mutation type.
- [ ] Saving then reloading reproduces the exact same page tree.
- [ ] The preview output matches the canvas with no editor-only elements leaking in.

## Common Pitfalls

- Storing the page as rendered HTML instead of a structured tree, making editing and undo intractable.
- Mutating the tree in place, so undo/redo and change detection become unreliable.
- Allowing invalid drops (a container into its own descendant) that create cycles and crash rendering.
- Coupling the property editor to concrete component code, so adding a component means editing the panel.
- Leaking editor-only wrappers (selection outlines, drag handles) into the serialized output.

## Resources

- [MDN: HTML Drag and Drop API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API) — the native drag interaction model.
- [dnd kit documentation](https://docs.dndkit.com/) — a modern, accessible dragging toolkit for building sortable/nestable UIs.
- [MDN: JSON](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON) — serializing and restoring the document tree.
- [Redux: Implementing Undo History](https://redux.js.org/usage/implementing-undo-history) — a clear model for undo/redo over state.
