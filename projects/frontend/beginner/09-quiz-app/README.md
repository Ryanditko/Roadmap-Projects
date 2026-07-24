# Quiz App

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build a multiple-choice quiz that walks a user through a set of questions one at a time, records their answers, and shows a scored results screen at the end. The questions live in a local array, so the work is all about flow and state: which question are we on, what has been answered, and what does the screen show right now. It is a clean introduction to conditional rendering and a tiny state machine — start, in-progress, and finished — plus the accessible rendering of radio-group choices where keyboard users can move and select without a mouse.

## Prerequisites

- HTML, CSS, and JavaScript fundamentals
- Array iteration and index tracking
- Conditional rendering (show one view based on state)
- Accessible radio groups and buttons

## Learning Objectives

By the end, you should be able to:

- Model quiz progress as explicit state (current index, answers, phase)
- Render one question at a time and move forward/back through the set
- Compute a score from recorded answers, not from the DOM
- Show a results screen with a per-question review
- Build an accessible radio group with keyboard selection and clear focus

## Functional Requirements

1. Questions are shown one at a time with their answer options.
2. The user selects one answer per question and can go to the next question.
3. A progress indicator shows the current position (e.g. "Question 3 of 10").
4. At the end, a results screen shows the score and which answers were right or wrong.
5. Answer options form an accessible radio group navigable by keyboard.
6. The user cannot advance past the last question without seeing results.
7. A restart option resets all state and returns to the first question.

## Suggested Milestones

1. **Milestone 1 — Render & answer:** Show one question with selectable options and record the choice.
2. **Milestone 2 — Navigation & progress:** Move between questions and display progress.
3. **Milestone 3 — Scoring & review:** Compute the score and render a results/review screen.

## Data & Interface Sketch

```text
Question
  id:       string
  prompt:   string
  options:  string[]
  answer:   number   (index of correct option)

Quiz state
  phase:    "start" | "playing" | "finished"
  index:    number
  answers:  (number | null)[]   (one slot per question)

Layout (playing)
+------------------------------------------+
| Question 3 of 10        [====------]     |
+------------------------------------------+
| What does CSS stand for?                 |
|  ( ) Cascading Style Sheets              |
|  ( ) Computer Style System               |
|  ( ) Creative Styling Syntax             |
+------------------------------------------+
| [ Back ]                     [ Next ]    |
+------------------------------------------+
```

## Stretch Goals

- Add an optional per-question countdown timer that auto-advances on expiry.
- Randomize question and option order on each run.
- Support multiple quizzes/categories chosen from a start screen.
- Persist a local high score across sessions with `localStorage`.

## Definition of Done

- [ ] Exactly one question shows at a time with its options.
- [ ] Progress accurately reflects the current question.
- [ ] The final score matches the recorded answers.
- [ ] Options are a keyboard-navigable radio group with visible focus.
- [ ] Restart fully resets state back to the first question.

## Common Pitfalls

- Reading the selected answer from the DOM instead of tracking it in state.
- Off-by-one errors in the progress counter or the last-question boundary.
- Using clickable `<div>`s for options, breaking keyboard and screen-reader use.
- Recomputing score incorrectly when a user changes an earlier answer.
- Not resetting every piece of state on restart, leaking the previous run.

## Resources

- [MDN: <input type="radio">](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/radio) — grouping and default keyboard behavior.
- [WAI-ARIA: Radio Group pattern](https://www.w3.org/WAI/ARIA/apg/patterns/radio/) — accessible custom radio groups.
- [MDN: Conditional rendering with JavaScript](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/conditionals) — switching views by state.
- [web.dev: Learn Accessibility — Focus](https://web.dev/learn/accessibility/focus) — managing and showing focus.
