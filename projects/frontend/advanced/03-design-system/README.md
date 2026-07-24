# Full Design System (component library)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build the shared visual foundation that many product teams consume — the layer behind Material, Carbon, and Polaris. A real design system is more than a folder of components: it is a token layer (colors, spacing, type) that feeds themeable, accessible components, published as a versioned package with living documentation. The engineering challenge is governance at scale. How do you evolve a button used by forty screens without breaking any of them? How do you catch a one-pixel visual regression before a consumer does? This project treats the system as a product with its own users, its own release process, and its own contract — semantic versioning, visual regression tests, and docs that never drift from the code.

## Prerequisites

- Solid component-authoring experience in one framework
- Understanding of CSS custom properties and the cascade
- Familiarity with a package registry and semantic versioning
- Comfort setting up a build that emits a distributable library

## Learning Objectives

By the end, you should be able to:

- Model a design-token layer that themes propagate through, decoupled from components
- Author accessible, composable components with well-typed, minimal APIs
- Publish a versioned package and communicate breaking changes via semver + changelog
- Catch visual and accessibility regressions automatically before release
- Maintain documentation that is generated from, and stays in sync with, the source

## Functional Requirements

1. Design tokens (color, spacing, typography, radius) must be defined once and consumed by all components.
2. At least two themes (e.g. light/dark) must switch purely by swapping token values, with no component code changes.
3. Every interactive component must be keyboard-operable and expose correct roles/ARIA.
4. The library must build into a tree-shakeable, versioned package that a separate app can install and import.
5. Documentation must render live, interactive examples of each component and its props.
6. A visual regression test must fail the build when a component's rendered output changes unexpectedly.
7. Breaking changes must bump the major version and be recorded in a human-readable changelog.

## Suggested Milestones

1. **Milestone 1 — Tokens & primitives:** Define the token layer and a few base components consuming it.
2. **Milestone 2 — Theming & a11y:** Add theme switching and make components keyboard- and screen-reader-friendly.
3. **Milestone 3 — Docs & playground:** Stand up Storybook (or equivalent) with interactive prop controls.
4. **Milestone 4 — Release pipeline:** Add visual regression + a11y checks and a versioned publish flow.

## Data & Interface Sketch

```text
        ┌─────────────────────────────────────────────┐
        │              Design tokens                   │
        │  color.* · space.* · font.* · radius.*       │
        └───────────────┬─────────────────────────────┘
                        │ CSS variables / theme object
        ┌───────────────▼─────────────────────────────┐
        │   Primitives: Box, Text, Icon, Stack          │
        └───────────────┬─────────────────────────────┘
        ┌───────────────▼─────────────────────────────┐
        │   Components: Button, Input, Modal, Table     │
        └───────────────┬─────────────────────────────┘
        ┌───────┴────────┐        ┌────────────────────┐
        │  Docs (stories) │        │  npm package (semver)│
        └────────────────┘        └────────────────────┘

Token flow:   token -> theme -> component (never hard-coded hex)
Versioning:   patch=fix · minor=additive · major=breaking API/visual
Gates:        visual regression snapshot + axe a11y scan per PR
```

## Stretch Goals

- Add a token pipeline (e.g. Style Dictionary) that emits CSS, JS, and native formats from one source.
- Generate an accessibility report per component and publish it alongside the docs.
- Support a runtime theming API so consumers can brand the system without a rebuild.
- Add a "deprecations" mechanism that warns in dev when a soon-to-be-removed prop is used.

## Definition of Done

- [ ] Switching theme changes the whole UI by swapping tokens, with zero component edits.
- [ ] A consuming app installs the package and imports only the components it uses (verified by bundle size).
- [ ] Every interactive component passes keyboard-only operation and an automated a11y scan.
- [ ] An intentional visual change fails the regression suite until the snapshot is reviewed and updated.
- [ ] The changelog and version reflect the nature of each change (patch/minor/major).

## Common Pitfalls

- Hard-coding colors and spacing in components instead of referencing tokens, breaking theming.
- Over-engineering component APIs with dozens of props instead of favoring composition.
- Letting docs drift from code by writing them by hand rather than generating from source.
- Publishing breaking changes as minor versions, silently breaking downstream apps.
- Treating accessibility as a later pass rather than a per-component acceptance criterion.

## Resources

- [Storybook documentation](https://storybook.js.org/docs) — the standard for building and documenting components in isolation.
- [Design Tokens Community Group format](https://tr.designtokens.org/format/) — the emerging standard for portable design tokens.
- [WAI-ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/) — authoritative patterns for accessible components.
- [Semantic Versioning](https://semver.org/) — the contract for communicating change through version numbers.
