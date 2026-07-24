# Image Gallery

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build a responsive image gallery: a grid of thumbnails that opens a larger image in a lightbox overlay when clicked, with previous/next navigation. The grid teaches responsive layout with CSS Grid; the lightbox teaches something harder and more valuable — building an accessible modal. A real lightbox must trap keyboard focus, close on Escape, restore focus when it closes, and be operable with arrow keys. Add lazy image loading and you have a project that touches layout, interaction, and performance in one compact package.

## Prerequisites

- HTML, CSS, and JavaScript fundamentals
- CSS Grid for responsive layouts
- DOM event handling including keyboard events
- A folder of sample images or a placeholder image service

## Learning Objectives

By the end, you should be able to:

- Build a responsive grid that adapts columns to the viewport with `auto-fill`/`minmax`
- Implement an accessible modal/lightbox with focus trapping and restoration
- Navigate images with previous/next buttons and arrow keys
- Lazy-load offscreen images to reduce initial load
- Provide meaningful `alt` text and keyboard-operable controls

## Functional Requirements

1. Images display in a responsive grid that reflows by viewport width.
2. Clicking a thumbnail opens a lightbox showing the full-size image.
3. The lightbox supports previous/next navigation via buttons and arrow keys.
4. Escape closes the lightbox and focus returns to the triggering thumbnail.
5. While the lightbox is open, keyboard focus stays trapped within it.
6. Offscreen images load lazily rather than all at once on page load.
7. Every image has descriptive `alt` text, and controls are labelled.

## Suggested Milestones

1. **Milestone 1 — Responsive grid:** Render thumbnails in a grid that adapts to screen width.
2. **Milestone 2 — Lightbox:** Open a full image overlay with previous/next controls.
3. **Milestone 3 — A11y & performance:** Add focus trap, Escape, keyboard nav, and lazy loading.

## Data & Interface Sketch

```text
Image
  id:      string
  src:     string   (full)
  thumb:   string   (small)
  alt:     string
  width:   number
  height:  number

Lightbox state
  isOpen:  boolean
  index:   number    (current image)

Grid (auto-fill columns)        Lightbox overlay
+-----+-----+-----+-----+       +------------------------+
| img | img | img | img |       |  [X]                   |
+-----+-----+-----+-----+       |   <   [ image ]   >    |
| img | img | img | img |  -->  |                        |
+-----+-----+-----+-----+       |   3 / 12   caption     |
                                +------------------------+
```

## Stretch Goals

- Add swipe gestures for touch devices to move between images.
- Add a thumbnail strip inside the lightbox showing position in the set.
- Add a slideshow mode that auto-advances, pausable and respecting reduced motion.
- Use `IntersectionObserver` for lazy loading instead of the `loading` attribute.

## Definition of Done

- [ ] The grid reflows without overflow from mobile to desktop.
- [ ] The lightbox opens, navigates, and closes by both mouse and keyboard.
- [ ] Focus is trapped while open and restored to the trigger on close.
- [ ] Images below the fold do not load until needed.
- [ ] All images carry appropriate `alt` text.

## Common Pitfalls

- Building the lightbox as a plain overlay that lets focus and Tab escape behind it.
- Not restoring focus on close, so keyboard users lose their place.
- Forgetting to set explicit image dimensions, causing layout shift as images load.
- Loading full-resolution images as thumbnails, wasting bandwidth.
- Making the close and nav controls non-focusable `<div>`s.

## Resources

- [MDN: CSS Grid Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout) — responsive grids with `minmax` and `auto-fill`.
- [MDN: Lazy loading](https://developer.mozilla.org/en-US/docs/Web/Performance/Lazy_loading) — deferring offscreen images.
- [WAI-ARIA: Dialog (Modal) pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/) — focus trap and keyboard behavior.
- [web.dev: prefers-reduced-motion](https://web.dev/articles/prefers-reduced-motion) — respecting motion preferences.
