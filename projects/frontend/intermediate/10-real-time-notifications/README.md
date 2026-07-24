# Real-time Notifications UI (Toasts)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a notification (toast) system: transient messages of different types — info, success, warning, error — that stack, auto-dismiss after a timer, and can be dismissed early or carry an action button. It is the kind of feature every app grows, and it is deceptively tricky. Timers must pause when the user hovers so a message is not yanked away mid-read, dismissing one toast must not disturb the countdown of its neighbours, and — the part most implementations get wrong — a screen reader has to hear an error without the toast stealing focus. You will build it as a small provider/queue that any component can push into, keeping timing logic out of the visual components.

## Prerequisites

- Comfort with component state, effects, and cleanup in a framework (React, Vue, Svelte, or Angular)
- Understanding of timers (`setTimeout`) and the importance of clearing them
- Familiarity with a global state pattern (context/store) for a cross-app queue
- Awareness of ARIA live regions and `role="status"` / `role="alert"`
- Basic CSS transitions and `prefers-reduced-motion`

## Learning Objectives

By the end, you should be able to:

- Design a notification queue as a provider any component can dispatch into
- Manage per-toast auto-dismiss timers, including pause-on-hover and resume
- Distinguish notification severities visually and semantically
- Announce notifications to assistive technology without hijacking focus
- Cap the number of visible toasts and handle overflow gracefully
- Clean up timers on unmount and on manual dismiss to avoid leaks

## Functional Requirements

1. Any part of the app must be able to trigger a notification via a shared dispatch function.
2. Notifications must support at least four types (info, success, warning, error) with distinct styling.
3. Each toast must auto-dismiss after a configurable timeout and be dismissible early via a close button.
4. Hovering (or focusing) a toast must pause its dismiss timer; leaving must resume it.
5. Toasts must stack, cap at a maximum visible count, and queue or collapse the overflow.
6. Notifications must be announced to screen readers — polite for info/success, assertive for errors — without moving focus.
7. A toast may carry an optional action button (e.g. "Undo") that runs a callback and dismisses.

## Suggested Milestones

1. **Milestone 1 — Queue & render:** Build the provider and a dispatch API; render a stack of typed toasts.
2. **Milestone 2 — Timers:** Add auto-dismiss with pause-on-hover, resume, and manual close.
3. **Milestone 3 — Overflow & actions:** Cap visible toasts, handle overflow, and add action buttons.
4. **Milestone 4 — Accessibility:** Wire live regions and correct roles, verify no focus stealing.

## Data & Interface Sketch

```text
Layout (top-right stack)
  ┌───────────────────────────────┐
  │ ✓ Saved successfully      [✕] │  ← success, polite
  ├───────────────────────────────┤
  │ ⚠ Session expiring soon   [✕] │  ← warning
  ├───────────────────────────────┤
  │ ✕ Upload failed  [Retry]  [✕] │  ← error, assertive, action
  └───────────────────────────────┘
  (+2 more…)   ← overflow indicator when over the cap

State (provider)
  toasts: Toast[]
  Toast {
    id, type: 'info'|'success'|'warning'|'error',
    message, action?: { label, onClick },
    duration: number, remaining: number, paused: boolean
  }
  MAX_VISIBLE = 4

Dispatch API
  notify({ type, message, duration?, action? }) -> id
  dismiss(id)

Accessibility
  container has aria-live: "polite" (info/success) region +
  a separate role="alert" region for errors; focus stays put
```

## Stretch Goals

- Let each toast pick its own position (corner) and animate in/out from that edge.
- Group or de-duplicate identical rapid-fire notifications with a count badge.
- Add a persistent "notification center" that keeps a history of dismissed toasts.
- Add a subtle progress bar showing the remaining time before auto-dismiss.
- Support promise-based toasts ("loading → success/error") for async actions.

## Definition of Done

- [ ] Any component can dispatch a notification through the shared provider.
- [ ] Toasts auto-dismiss, pause on hover/focus, and resume on leave.
- [ ] The visible count is capped and overflow is handled without breaking layout.
- [ ] Errors are announced assertively and info politely, with focus never stolen.
- [ ] Timers are cleared on dismiss and unmount — no callbacks fire after removal.

## Common Pitfalls

- Storing timer IDs loosely and never clearing them, firing dismiss on already-removed toasts.
- Rebuilding all timers whenever the list changes, resetting every countdown on each new toast.
- Auto-dismissing errors on a short timer before the user can read or act on them.
- Moving focus to a toast, trapping keyboard users and interrupting their task.
- Relying on color alone to signal severity, failing colorblind users and contrast checks.

## Resources

- [MDN: ARIA live regions](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions) — polite vs assertive announcements.
- [W3C ARIA APG: Alert pattern](https://www.w3.org/WAI/ARIA/apg/patterns/alert/) — the semantics of a notification.
- [MDN: setTimeout / clearTimeout](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout) — the timer lifecycle to manage.
- [web.dev: prefers-reduced-motion](https://web.dev/articles/prefers-reduced-motion) — animating toasts responsibly.
- [Inclusive Components: Toast/notifications](https://inclusive-components.design/notifications/) — an accessible design walkthrough.
