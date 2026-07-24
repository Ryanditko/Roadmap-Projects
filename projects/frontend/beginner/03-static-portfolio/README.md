# Static Portfolio Website

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build a personal portfolio site that presents who you are, what you have built, and how to reach you — all as static, content-first pages with no backend. The point is not flashy animation; it is writing clean, semantic HTML and a responsive CSS layout that reads well from a 320px phone to a wide desktop. You will structure real sections (intro, projects, skills, contact), make the navigation usable by keyboard and screen reader, and let the content drive the design rather than the other way around. Done well, this becomes something you actually deploy and link on your résumé.

## Prerequisites

- HTML fundamentals and the difference between semantic and generic elements
- CSS box model, Flexbox, and CSS Grid basics
- An understanding of media queries and relative units (`rem`, `%`, `vw`)
- A code editor and a browser with dev tools

## Learning Objectives

By the end, you should be able to:

- Structure a page with semantic landmarks (`header`, `nav`, `main`, `section`, `footer`)
- Build a layout that reflows gracefully across screen sizes without horizontal scroll
- Use Grid and Flexbox for the right jobs (page skeleton vs. component alignment)
- Write accessible navigation with a logical heading order and visible focus states
- Optimize images with responsive `srcset`/`sizes` and meaningful `alt` text

## Functional Requirements

1. The site has clearly separated sections: intro/hero, about, projects, skills, and contact.
2. A navigation bar links to each section and works with keyboard and touch.
3. The layout is fully responsive with no horizontal overflow at 320px width.
4. Each project entry shows a title, short description, and a link to code or a live demo.
5. All images have descriptive `alt` text; decorative images use empty `alt`.
6. Headings follow a single, logical order (`h1` → `h2` → `h3`) with no skipped levels.
7. Color contrast meets WCAG AA for body text and interactive elements.

## Suggested Milestones

1. **Milestone 1 — Structure & content:** Write the semantic HTML for all sections with real placeholder content.
2. **Milestone 2 — Responsive layout:** Style with Grid/Flexbox and add media queries for mobile, tablet, and desktop.
3. **Milestone 3 — Polish & a11y:** Add focus states, contrast fixes, responsive images, and smooth in-page navigation.

## Data & Interface Sketch

```text
Page landmarks
  header > nav (logo + section links)
  main
    section#hero    (name, role, one-line pitch, CTA)
    section#about   (short bio)
    section#projects (grid of project cards)
    section#skills  (grouped skill tags)
    section#contact (email + social links)
  footer (copyright, back-to-top)

Project card
  title:    string
  summary:  string
  tags:     string[]
  repoUrl / liveUrl: string

Desktop grid           Mobile (stacked)
+----+----+----+        +-----------+
| c1 | c2 | c3 |        |    c1     |
+----+----+----+   -->  +-----------+
| c4 | c5 | c6 |        |    c2     |
+----+----+----+        +-----------+
```

## Stretch Goals

- Add a light/dark theme toggle that respects `prefers-color-scheme`.
- Filter projects by tag/category without a page reload.
- Add subtle scroll-reveal animations gated behind `prefers-reduced-motion`.
- Deploy to a free static host and wire up a custom domain.

## Definition of Done

- [ ] Every section is reachable from the nav by mouse, touch, and keyboard.
- [ ] The layout has zero horizontal scroll from 320px up to desktop widths.
- [ ] All content images have appropriate `alt` text.
- [ ] Heading levels are ordered with no skips, and there is exactly one `h1`.
- [ ] Text and interactive colors pass WCAG AA contrast.

## Common Pitfalls

- Wrapping everything in `<div>`s instead of semantic landmarks, hurting screen-reader navigation.
- Fixed pixel widths that force horizontal scrolling on small screens.
- Skipping heading levels (`h1` straight to `h4`) for visual size instead of using CSS.
- Low-contrast "designer grey" text that fails accessibility checks.
- Shipping huge unoptimized images that tank load time on mobile.

## Resources

- [MDN: HTML elements reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Element) — choosing the right semantic element.
- [web.dev: Learn Responsive Design](https://web.dev/learn/design) — building layouts that adapt.
- [web.dev: Learn Accessibility](https://web.dev/learn/accessibility) — landmarks, headings, and contrast.
- [MDN: Responsive images](https://developer.mozilla.org/en-US/docs/Web/HTML/Responsive_images) — `srcset` and `sizes`.
