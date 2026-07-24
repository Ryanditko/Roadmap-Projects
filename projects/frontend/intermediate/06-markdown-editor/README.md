# Markdown Editor (Live Preview)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a split-pane markdown editor: raw markdown on the left, a rendered HTML preview on the right, updating as the user types. It sounds simple until you notice the traps — rendering untrusted markdown is an XSS vector, re-parsing on every keystroke stalls on large documents, and a naive `innerHTML` write throws away the user's cursor and scroll position. The real lesson here is turning text into safe, sanitized HTML efficiently, wiring a formatting toolbar and keyboard shortcuts that edit the textarea intelligently, and keeping the editor responsive as the document grows. You will lean on a real markdown parser rather than reinventing one.

## Prerequisites

- Comfort with component state and controlled inputs in a framework (React, Vue, Svelte, or Angular)
- Understanding of the DOM and why setting `innerHTML` from user input is dangerous
- Familiarity with textarea selection APIs (`selectionStart`/`selectionEnd`)
- Basic knowledge of debouncing and why it helps typing performance
- A markdown parser (marked, markdown-it) and a sanitizer (DOMPurify) of your choice

## Learning Objectives

By the end, you should be able to:

- Parse markdown to HTML with a maintained library rather than hand-rolled regex
- Sanitize rendered HTML so untrusted input cannot inject scripts
- Debounce or throttle parsing to keep typing smooth in a large document
- Manipulate a textarea's selection to implement bold/italic/link toolbar actions
- Wire keyboard shortcuts that map to formatting commands
- Persist a draft so work is not lost on an accidental reload

## Functional Requirements

1. The editor must show a source pane and a live preview pane that updates as the user types.
2. Markdown must be rendered with a real parser supporting headings, lists, links, code blocks, and emphasis.
3. Rendered HTML must be sanitized before insertion so `<script>` and event-handler attributes cannot execute.
4. A toolbar must apply formatting (bold, italic, link, code) to the current selection, wrapping or inserting correctly.
5. At least three keyboard shortcuts (e.g. bold, italic, save) must trigger the matching action.
6. Parsing must be debounced or otherwise bounded so a large document does not freeze on each keystroke.
7. The current document must persist locally and restore on reload.

## Suggested Milestones

1. **Milestone 1 — Parse & preview:** Wire the source pane to a parser and render sanitized HTML in the preview.
2. **Milestone 2 — Toolbar:** Add selection-aware formatting buttons that wrap or insert markdown syntax.
3. **Milestone 3 — Shortcuts & perf:** Add keyboard shortcuts and debounce parsing for large documents.
4. **Milestone 4 — Persistence:** Auto-save the draft and restore it on load.

## Data & Interface Sketch

```text
Layout
  ┌───────────────────────────────────────────────┐
  │ Toolbar: [B] [I] [link] [code] [H1] [list]     │
  ├──────────────────────┬────────────────────────┤
  │ # Source (textarea)  │ Preview (sanitized)     │
  │ **bold** text        │ <strong>bold</strong>…  │
  │ - item               │ • item                  │
  └──────────────────────┴────────────────────────┘

State
  source:   string          // raw markdown
  html:     string          // derived: sanitize(parse(source)), debounced
  draftKey: 'md-editor:draft'

Formatting action (pure over selection)
  applyWrap(source, selStart, selEnd, marker)
    -> { text, newSelStart, newSelEnd }
    // e.g. marker "**" turns "cat" into "**cat**"

Pipeline
  keystroke → debounce(150ms) → parse(md) → sanitize(html) → render
```

## Stretch Goals

- Add syntax highlighting inside fenced code blocks.
- Sync-scroll the source and preview panes proportionally.
- Generate a table-of-contents outline from the headings.
- Export the rendered document to a standalone HTML file.
- Add an undo/redo history layer above the textarea's native one.

## Definition of Done

- [ ] Typing markdown updates a correctly rendered preview.
- [ ] A pasted `<script>` or `onerror` attribute is stripped and never executes.
- [ ] Toolbar buttons wrap the current selection rather than replacing or clearing it.
- [ ] Typing in a long document stays responsive (parsing is debounced/bounded).
- [ ] A draft survives a reload without data loss.

## Common Pitfalls

- Writing parser output straight into `innerHTML` without sanitizing — the classic stored-XSS hole.
- Re-parsing synchronously on every keystroke, freezing the tab on a large file.
- Replacing the whole textarea value on each format, blowing away the cursor and undo stack.
- Rolling your own markdown regex and drowning in edge cases a library already handles.
- Sanitizing on the source instead of the rendered HTML, breaking legitimate markdown.

## Resources

- [MDN: Element.innerHTML security considerations](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML#security_considerations) — why raw HTML injection is dangerous.
- [DOMPurify](https://github.com/cure53/DOMPurify) — a battle-tested HTML sanitizer.
- [marked documentation](https://marked.js.org/) — a fast, well-maintained markdown parser.
- [CommonMark specification](https://spec.commonmark.org/) — the reference for what markdown means.
- [MDN: HTMLTextAreaElement.setSelectionRange()](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/setSelectionRange) — manipulating selection for toolbar actions.
