# Timer App

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build a timer app with two modes: a stopwatch that counts up and a countdown that counts down to zero and alerts. It looks simple, but it teaches the single most misunderstood thing about time in the browser — you cannot trust `setInterval` to be accurate. Intervals drift, and they slow down or pause entirely in background tabs. The correct approach is to store a start timestamp and compute elapsed time from the real clock on each tick, using the interval only to trigger repaints. Get that right and your timer stays accurate even after the tab was hidden for a minute.

## Prerequisites

- HTML, CSS, and JavaScript fundamentals
- `setInterval`, `clearInterval`, and `Date.now()`
- Basic state modelling (running vs. paused vs. stopped)
- Formatting numbers into `MM:SS` strings

## Learning Objectives

By the end, you should be able to:

- Track elapsed time from a real timestamp rather than accumulating interval ticks
- Implement start, pause, resume, and reset without drift
- Build both stopwatch (count up) and countdown (count down) modes
- Format time consistently and update the display smoothly
- Fire an accessible alert/notification when a countdown reaches zero

## Functional Requirements

1. The stopwatch counts up from zero and can be paused, resumed, and reset.
2. The countdown accepts a duration, counts down to zero, and signals completion.
3. Elapsed/remaining time is computed from real timestamps, staying accurate after a background tab.
4. Time is displayed in a clear `MM:SS` (or `HH:MM:SS`) format.
5. Pause preserves the exact elapsed time; resume continues from it.
6. Reset returns to the initial state with controls in the correct enabled/disabled states.
7. Countdown completion is announced accessibly, not by color alone.

## Suggested Milestones

1. **Milestone 1 — Stopwatch:** Count up from a start timestamp with start/pause/reset.
2. **Milestone 2 — Countdown:** Accept a duration and count down, signalling zero.
3. **Milestone 3 — Accuracy & polish:** Verify no drift across tab backgrounding; add the completion alert.

## Data & Interface Sketch

```text
Timer state
  mode:       "stopwatch" | "countdown"
  status:     "idle" | "running" | "paused"
  startedAt:  number | null   (Date.now() at last start)
  baseElapsed: number         (ms accumulated before current run)
  durationMs: number          (countdown target)

elapsed = baseElapsed + (running ? Date.now() - startedAt : 0)

Layout
+------------------------------------------+
|            00:00:00                      |  <- display
+------------------------------------------+
|   [ Start ]  [ Pause ]  [ Reset ]        |
+------------------------------------------+
|   Mode: (o) Stopwatch  ( ) Countdown     |
+------------------------------------------+
```

## Stretch Goals

- Add lap timing to the stopwatch, recording split times in a list.
- Add Pomodoro presets that cycle work and break countdowns.
- Use the Notification API to alert when the tab is not focused.
- Add a circular progress ring that fills as the countdown runs (respect reduced motion).

## Definition of Done

- [ ] Elapsed time is derived from timestamps and stays accurate after backgrounding.
- [ ] Start, pause, resume, and reset all behave correctly with matching control states.
- [ ] Both stopwatch and countdown modes work independently.
- [ ] Countdown completion is announced without relying on color alone.
- [ ] Time is always formatted consistently, with no negative or `NaN` values.

## Common Pitfalls

- Incrementing a counter each `setInterval` tick, which drifts and stalls in background tabs.
- Forgetting to `clearInterval`, leaking timers and stacking updates.
- Not preserving elapsed time across pause/resume, resetting the clock accidentally.
- Signalling completion only with a color change, invisible to some users.
- Displaying negative time when the countdown overshoots zero on a slow tick.

## Resources

- [MDN: setInterval()](https://developer.mozilla.org/en-US/docs/Web/API/setInterval) — its guarantees and its limits.
- [MDN: Date.now()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/now) — the timestamp your accuracy depends on.
- [MDN: Page Visibility API](https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API) — knowing when a tab is hidden.
- [MDN: ARIA live regions](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions) — announcing completion accessibly.
