# Calculator UI

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build a working calculator with a keypad, a display, and the four basic operations. The visual layout is the easy half; the real lesson is modelling calculator behaviour as a small state machine. A user taps `7`, `+`, `3`, `=` — each key means something different depending on what came before. You will track the current entry, the pending operator, and the accumulated value, and decide what every button does in every state. Get that model right and edge cases like chaining operations or pressing `=` twice stop being surprises.

## Prerequisites

- Basic HTML, CSS, and JavaScript
- CSS Grid or Flexbox for laying out a button pad
- Comfort with `switch`/conditional logic and number parsing
- A code editor and a browser with dev tools

## Learning Objectives

By the end, you should be able to:

- Represent interactive behaviour as explicit state rather than reading the display back as data
- Handle operator precedence for a simple left-to-right calculator and chained operations
- Format numeric output and avoid raw floating-point artefacts
- Support both mouse/touch and physical keyboard input for the same actions
- Guard against invalid states like multiple decimals or division by zero

## Functional Requirements

1. The calculator performs addition, subtraction, multiplication, and division.
2. The display shows the number being entered and the result after `=`.
3. Pressing an operator after a result uses that result as the new left operand (chaining).
4. A clear button resets all state; a backspace removes the last entered digit.
5. Only one decimal point is allowed per number.
6. Division by zero shows a clear error state rather than `Infinity` or `NaN`.
7. Number and operator keys on the physical keyboard mirror the on-screen buttons.

## Suggested Milestones

1. **Milestone 1 — Entry & display:** Build the keypad and show digits, decimals, and backspace.
2. **Milestone 2 — Single operation:** Store an operator and left operand, compute on `=`.
3. **Milestone 3 — Chaining & guards:** Chain operations, handle clear, decimals, and divide-by-zero.

## Data & Interface Sketch

```text
State
  current:   string    (digits being typed, e.g. "12.5")
  operator:  "+"|"-"|"*"|"/"|null
  accumulator: number|null
  justEvaluated: boolean

Layout (4-column grid)
+-------------------------------+
|            123.45             |  <- display
+-------------------------------+
|  C  | +/- |  %  |   /         |
|  7  |  8  |  9  |   *         |
|  4  |  5  |  6  |   -         |
|  1  |  2  |  3  |   +         |
|    0      |  .  |   =         |
+-------------------------------+
```

## Stretch Goals

- Add a running history/tape of previous calculations.
- Add memory keys (M+, M-, MR, MC).
- Support percentage relative to the current accumulator.
- Add a keyboard-only mode with visible focus indicators on each key.

## Definition of Done

- [ ] All four operations produce correct results, including chained sequences.
- [ ] The display never shows `NaN`, `Infinity`, or `undefined`.
- [ ] Clear fully resets state; backspace edits the current entry only.
- [ ] Keyboard and on-screen buttons behave identically.
- [ ] Only one decimal point can be entered per number.

## Common Pitfalls

- Using the display text as your source of truth instead of a separate state model.
- Concatenating strings for math and getting `"1" + "2" = "12"` instead of `3`.
- Ignoring floating-point rounding, so `0.1 + 0.2` shows `0.30000000000000004`.
- Forgetting the "just evaluated" flag, so the next digit appends to the result instead of starting fresh.
- Leaving buttons as non-focusable `<div>`s, breaking keyboard access.

## Resources

- [MDN: Number.prototype.toFixed()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/toFixed) — controlling decimal display.
- [MDN: CSS Grid Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout) — laying out the keypad.
- [0.30000000000000004.com](https://0.30000000000000004.com/) — why floating-point math misbehaves.
- [MDN: KeyboardEvent.key](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key) — mapping physical keys to actions.
