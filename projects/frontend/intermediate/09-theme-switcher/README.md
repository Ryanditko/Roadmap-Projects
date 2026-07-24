# Theme Switcher (Light / Dark / System)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a theming system that lets a user switch between light, dark, and "follow the system" modes, with the choice remembered across visits. The demo looks trivial — flip some colors — but doing it well touches subtler problems: driving the whole UI from CSS custom properties so a single attribute swap re-themes everything, respecting the OS preference via `prefers-color-scheme`, and eliminating the "flash of the wrong theme" that happens when the page paints before your script runs. You will also make sure both palettes actually pass contrast requirements, because a dark theme that fails WCAG is just a different kind of broken.

## Prerequisites

- Solid CSS fundamentals, especially custom properties (variables) and selectors
- Comfort with basic DOM scripting (reading/setting attributes on `<html>`)
- Understanding of `localStorage` for persistence
- Familiarity with media queries, specifically `prefers-color-scheme`
- Awareness of color contrast and WCAG contrast ratios

## Learning Objectives

By the end, you should be able to:

- Architect a theme with CSS custom properties so one attribute controls all colors
- Support three modes — light, dark, and system — with system tracking the OS live
- Persist the user's explicit choice and fall back to the system preference otherwise
- Prevent the flash of incorrect theme (FOUC) with an inline pre-paint script
- Verify both palettes meet WCAG AA contrast for text and interactive elements
- Respect `prefers-reduced-motion` when transitioning between themes

## Functional Requirements

1. The UI must be themeable entirely through CSS custom properties keyed off a `data-theme` attribute (or `class`) on the root.
2. A control must switch between light, dark, and system modes.
3. In system mode, the theme must follow the OS setting and update live if the OS preference changes.
4. An explicit user choice must persist across reloads; with no choice, the app defaults to system.
5. The correct theme must be applied before first paint, with no visible flash of the wrong colors.
6. Both light and dark palettes must meet WCAG AA contrast for body text and controls.
7. Theme transitions must be disabled or reduced when `prefers-reduced-motion: reduce` is set.

## Suggested Milestones

1. **Milestone 1 — Token palette:** Define light and dark palettes as CSS custom properties toggled by a root attribute.
2. **Milestone 2 — Switcher:** Add a control to set light/dark/system and apply it to the root.
3. **Milestone 3 — Persist & system:** Persist the choice, default to system, and react to live OS changes.
4. **Milestone 4 — Polish:** Kill the FOUC with a pre-paint script and verify contrast and reduced motion.

## Data & Interface Sketch

```text
Root attribute drives everything
  <html data-theme="dark">
  :root                 { --bg: #fff; --fg: #111; --accent: #2563eb; }
  [data-theme="dark"]   { --bg: #111; --fg: #eee; --accent: #60a5fa; }
  body { background: var(--bg); color: var(--fg); }

Resolution logic
  choice: 'light' | 'dark' | 'system'      (persisted in localStorage)
  system: matchMedia('(prefers-color-scheme: dark)').matches
  effective = choice === 'system' ? (system ? 'dark' : 'light') : choice

Live OS updates
  matchMedia('(prefers-color-scheme: dark)')
    .addEventListener('change', reapplyIfSystemMode)

Pre-paint (avoid FOUC): an inline <head> script sets data-theme
  from localStorage BEFORE the stylesheet-driven body renders
```

## Stretch Goals

- Add more themes (e.g. high-contrast, sepia) selectable from a dropdown.
- Add a scheduled auto-switch (dark after sunset) using local time.
- Offer an accent-color picker that writes a custom property.
- Export/import a theme configuration as JSON.
- Animate the toggle with a view transition, gated on reduced-motion.

## Definition of Done

- [ ] Every color comes from a custom property; switching the root attribute re-themes the whole app.
- [ ] Light, dark, and system modes all work, and system tracks a live OS change.
- [ ] The chosen mode persists across reloads and defaults to system when unset.
- [ ] There is no flash of the wrong theme on first paint.
- [ ] Both palettes pass WCAG AA contrast and transitions respect reduced-motion.

## Common Pitfalls

- Hard-coding colors in components instead of referencing tokens, so half the UI ignores the theme.
- Reading `localStorage` in a late-running bundle, causing a visible flash before the theme applies.
- Treating "system" as a one-time read instead of subscribing to `matchMedia` change events.
- Shipping a dark palette that looks nice but fails contrast on secondary text.
- Applying a global `transition: all` that makes the whole page lurch on every theme change.

## Resources

- [MDN: Using CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties) — the foundation of tokenized theming.
- [MDN: prefers-color-scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme) — detecting the OS theme.
- [web.dev: prefers-color-scheme](https://web.dev/articles/prefers-color-scheme) — building a robust dark mode, including FOUC.
- [WebAIM: Contrast and color accessibility](https://webaim.org/articles/contrast/) — meeting WCAG contrast ratios.
- [MDN: prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion) — honoring motion preferences.
