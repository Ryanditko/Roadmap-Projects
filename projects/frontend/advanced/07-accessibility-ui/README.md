# Accessibility-First UI System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build a set of complex, interactive UI patterns — modal dialogs, comboboxes, tab panels, data grids, menus — where accessibility is the primary design constraint, not a retrofit. These are exactly the widgets that break for keyboard and screen-reader users when built naively, because the browser gives you no built-in semantics for them. You will implement them against the WAI-ARIA Authoring Practices: correct roles, managed focus, predictable keyboard interaction, and announcements that make state changes perceivable without sight. The measure of success is not a passing automated scan (those catch perhaps a third of issues) but a real user operating the whole interface with only a keyboard and a screen reader, never getting stuck.

## Prerequisites

- Solid HTML and DOM manipulation skills
- Understanding of semantic HTML and the accessibility tree
- Willingness to test with an actual screen reader (NVDA, VoiceOver, or Orca)
- Familiarity with focus, `tabindex`, and keyboard event handling

## Learning Objectives

By the end, you should be able to:

- Implement WAI-ARIA authoring patterns for complex widgets correctly
- Manage focus deliberately: focus trapping, restoration, and roving tabindex
- Use ARIA roles, states, and live regions to make dynamic changes perceivable
- Test with keyboard-only navigation and a real screen reader, not just automated tools
- Meet WCAG success criteria for contrast, focus visibility, and target size

## Functional Requirements

1. Every interactive widget must be fully operable with the keyboard alone, following its ARIA pattern's expected keys.
2. A modal dialog must trap focus while open and restore focus to the trigger on close.
3. Dynamic updates (validation errors, async results) must be announced via appropriate live regions.
4. Focus must always be visible and never lost to an off-screen or detached element.
5. Color contrast must meet WCAG AA for text and UI components.
6. Each widget must expose correct roles and states in the accessibility tree, verified in dev tools.
7. The interface must remain usable at 200% zoom and with reduced-motion preferences honored.

## Suggested Milestones

1. **Milestone 1 — Semantics & keyboard:** Build 2–3 widgets with correct roles and full keyboard operation.
2. **Milestone 2 — Focus management:** Add focus trapping/restoration and roving tabindex where the pattern requires it.
3. **Milestone 3 — Announcements:** Wire live regions for validation, loading, and async result messaging.
4. **Milestone 4 — Verification:** Test the whole flow with keyboard + screen reader and fix what automation missed.

## Data & Interface Sketch

```text
   Widget: Combobox (WAI-ARIA pattern)
   ┌──────────────────────────────────────────┐
   │ input  role=combobox  aria-expanded=?      │
   │        aria-controls=listbox-id            │
   │        aria-activedescendant=option-id     │
   └───────────────┬────────────────────────────┘
                   ▼ (opens)
   ┌──────────────────────────────────────────┐
   │ ul role=listbox                            │
   │   li role=option  aria-selected=?          │  ← ↑/↓ move, Enter selects
   └──────────────────────────────────────────┘

Focus model:   dialog=trap+restore · menu/grid=roving tabindex
Live regions:  status (polite) for results · alert (assertive) for errors

WCAG targets (AA):
  text contrast        >= 4.5:1  (>= 3:1 large text / UI)
  focus indicator      visible, >= 3:1 against background
  operable             100% keyboard, honors prefers-reduced-motion
  usable at 200% zoom  no loss of content or function
```

## Stretch Goals

- Add a skip-link and landmark structure so screen-reader users can jump between regions.
- Support high-contrast / forced-colors mode without losing meaning conveyed by color.
- Add an automated a11y check (axe) in CI as a floor, documenting what it cannot catch.
- Provide a written keyboard-shortcut reference discoverable from within the UI.

## Definition of Done

- [ ] Every widget is operable with keyboard only, following its ARIA pattern's key bindings.
- [ ] A screen reader announces widget role, state, and dynamic updates correctly.
- [ ] Opening and closing a dialog traps then restores focus predictably.
- [ ] All text and controls meet WCAG AA contrast, verified with a contrast tool.
- [ ] The interface works at 200% zoom and respects reduced-motion preferences.

## Common Pitfalls

- Adding ARIA roles to non-semantic elements while forgetting the keyboard behavior the role implies.
- Using `aria-label` to paper over a missing accessible structure instead of fixing the semantics.
- Trapping focus in a dialog but never restoring it, stranding the user after close.
- Overusing `aria-live="assertive"`, flooding the screen reader and drowning out important alerts.
- Shipping on a green automated scan alone, which misses most real usability barriers.

## Resources

- [WAI-ARIA Authoring Practices Guide (APG)](https://www.w3.org/WAI/ARIA/apg/) — reference patterns with expected keyboard behavior.
- [WebAIM: Introduction to Screen Readers](https://webaim.org/articles/screenreader_testing/) — how to test with assistive technology.
- [MDN: ARIA live regions](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions) — announcing dynamic changes.
- [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/) — the success criteria and how to meet them.
