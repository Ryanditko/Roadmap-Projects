# Login & Register Forms

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build the front end for authentication: a login form and a registration form with real, client-side validation. There is no backend — you simulate submission — so the lesson is entirely about form UX, which is where a huge amount of frontend craft lives. You will validate fields as the user types, show precise error messages next to the right inputs, disable submission until the form is valid, and do all of it accessibly so screen-reader users know exactly what went wrong. Getting validation timing and messaging right is the difference between a form people complete and one they abandon.

## Prerequisites

- HTML forms, inputs, and the `<label>` element
- JavaScript event handling (`input`, `submit`, `blur`)
- Regular expressions or built-in validation for pattern checks
- A basic grasp of why client validation is UX, not security

## Learning Objectives

By the end, you should be able to:

- Validate fields on the right events (on blur / on submit, not aggressively on first keystroke)
- Associate error messages with inputs using `aria-describedby` and `aria-invalid`
- Manage form state: values, touched fields, errors, and submit-disabled logic
- Build a password strength indicator and a show/hide password toggle
- Prevent double submission and give clear success feedback

## Functional Requirements

1. The registration form validates email format, password rules, and password confirmation match.
2. Errors appear next to the relevant field and are announced to assistive technology.
3. The submit button is disabled (or blocks submission) while the form is invalid.
4. A password strength indicator updates as the user types.
5. A show/hide toggle reveals or masks the password field.
6. The login form validates required fields and shows a single form-level error on "failure".
7. Submitting shows a loading state, then a success confirmation, preventing double submits.

## Suggested Milestones

1. **Milestone 1 — Structure & labels:** Build both forms with properly labelled, accessible inputs.
2. **Milestone 2 — Validation:** Add field validation, error messaging, and submit gating.
3. **Milestone 3 — Feedback & polish:** Add strength meter, show/hide, loading, and success states.

## Data & Interface Sketch

```text
Field state (per input)
  value:   string
  touched: boolean
  error:   string | null

Register form fields
  email             -> valid email pattern
  password          -> min length, mixed character rules
  confirmPassword   -> must equal password
  acceptTerms       -> must be checked

Layout (register)
+----------------------------------+
| Email    [ .................. ]  |
|          error text (aria)       |
| Password [ ............ ] [👁]    |
|          [====----] Medium       |
| Confirm  [ .................. ]  |
| [x] I accept the terms           |
| [        Create account       ]  |  <- disabled until valid
+----------------------------------+
```

## Stretch Goals

- Add a "forgot password" request form with its own validation.
- Persist a draft of the registration form to `sessionStorage`.
- Add inline "available/taken" username checking against a mock list.
- Support submit-on-Enter and full keyboard navigation with a logical tab order.

## Definition of Done

- [ ] Every input has a programmatically associated label.
- [ ] Errors are tied to inputs via `aria-describedby` and set `aria-invalid`.
- [ ] Submission is blocked while any field is invalid.
- [ ] The password toggle changes the input type without losing the value.
- [ ] A submit cannot fire twice, and success is clearly communicated.

## Common Pitfalls

- Validating on every keystroke from the first character, so users see errors before they finish typing.
- Using `placeholder` as a substitute for `<label>`, which disappears and fails accessibility.
- Showing a generic "invalid form" message instead of field-specific errors.
- Treating client validation as security — the real check must happen on a server.
- Forgetting to focus or announce errors, leaving screen-reader users stuck.

## Resources

- [MDN: Client-side form validation](https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation) — constraints and custom messages.
- [web.dev: Sign-in form best practices](https://web.dev/articles/sign-in-form-best-practices) — real-world form UX guidance.
- [MDN: aria-invalid](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-invalid) — signalling invalid fields to AT.
- [WAI: Form instructions & errors](https://www.w3.org/WAI/tutorials/forms/) — accessible form patterns.
