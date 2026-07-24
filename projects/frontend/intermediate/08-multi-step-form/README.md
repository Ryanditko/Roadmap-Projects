# Multi-Step Form (Wizard)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a multi-step form — a wizard — that splits a long form (say, account → profile → payment → review) across several screens with a progress indicator, per-step validation, and a final review before submit. A single giant form is easy; the wizard is harder because state now has to survive navigation between steps, a step must not let the user advance until its fields are valid, some steps may be conditional on earlier answers, and the whole thing has to stay accessible and recoverable if the user reloads mid-flow. This project is really about form state architecture and validation timing, not markup.

## Prerequisites

- Comfort with controlled form inputs and component state in a framework (React, Vue, Svelte, or Angular)
- Understanding of validation (field-level and step-level) and when to run it
- Familiarity with derived state (progress, "can advance?") from a single form model
- Basic knowledge of `localStorage` or session persistence
- Optional: a form/validation library (React Hook Form, Formik, Zod, Yup) versus hand-rolled

## Learning Objectives

By the end, you should be able to:

- Hold the entire form in one state model that persists across step navigation
- Validate a step before allowing forward navigation while permitting free backward movement
- Model conditional steps that appear or vanish based on earlier answers
- Build an accessible progress indicator and manage focus on step change
- Assemble a review step that reads back all collected data before submit
- Persist and restore progress so a reload does not lose entered data

## Functional Requirements

1. The form must span at least three steps with a visible progress indicator showing current and total steps.
2. Each step must validate its own fields, and "Next" must be blocked until the current step is valid.
3. "Back" must return to the previous step with all previously entered values intact.
4. At least one step must be conditional — shown or skipped based on a value from an earlier step.
5. A final review step must display all entered data and allow jumping back to edit any step.
6. On submit, the assembled payload must be validated as a whole before the request fires.
7. Form progress must persist so a reload restores the current step and entered values.

## Suggested Milestones

1. **Milestone 1 — Steps & navigation:** Render steps with next/back and a progress indicator over one state model.
2. **Milestone 2 — Validation gate:** Add per-step validation that gates forward navigation and shows errors.
3. **Milestone 3 — Conditional & review:** Add a conditional step and a review screen with edit links.
4. **Milestone 4 — Persist & submit:** Save progress, restore on reload, and validate the full payload on submit.

## Data & Interface Sketch

```text
Layout
  ①──②──③──④    ← progress: step 2 of 4 active
  ┌───────────────────────────────┐
  │ Step 2: Profile                │
  │ [ Name ...... ]  (error text)  │
  │ [ Bio  ...... ]                │
  ├───────────────────────────────┤
  │        [ Back ]   [ Next → ]   │  ← Next disabled until valid
  └───────────────────────────────┘

State (single model)
  currentStep: number
  data: { account: {...}, profile: {...}, payment?: {...} }
  errors: Record<field, string>
  visitedSteps: Set<number>

Step definition
  steps: {
    id, title,
    fields: string[],
    validate(data) -> Record<field,string>,   // empty = valid
    isVisible(data) -> boolean                 // conditional steps
  }[]

Persistence
  save data + currentStep to sessionStorage on each change; restore on load
```

## Stretch Goals

- Add a "save and continue later" link that resumes from a stored token.
- Animate step transitions while respecting `prefers-reduced-motion`.
- Show a summary sidebar that updates live as the user fills each step.
- Support deep-linking to a step via the URL, guarded by validation of prior steps.
- Add asynchronous validation for a field (e.g. username availability).

## Definition of Done

- [ ] Data entered in one step is still present after navigating away and back.
- [ ] "Next" is blocked with visible errors until the current step is valid.
- [ ] A conditional step appears or is skipped correctly based on earlier input.
- [ ] The review step reflects all data and links back to edit each step.
- [ ] A reload restores the current step and all entered values.

## Common Pitfalls

- Storing each step's data in a separate component that unmounts, losing values on navigation.
- Validating only on submit, so users reach the last step before learning step 1 was wrong.
- Letting a hidden conditional step's stale data leak into the final payload.
- Not moving focus to the new step's heading, stranding keyboard and screen-reader users.
- Blocking backward navigation behind validation — going back should always be free.

## Resources

- [MDN: Client-side form validation](https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation) — validation fundamentals.
- [web.dev: Sign-in form best practices](https://web.dev/articles/sign-in-form-best-practices) — multi-field form UX and autofill.
- [W3C ARIA APG: Focus management](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/) — moving focus on step change.
- [React Hook Form documentation](https://react-hook-form.com/) — a widely used form state/validation library.
- [MDN: Window.sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage) — persisting wizard progress.
