# E-commerce Frontend

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build the shopper-facing front end of an online store: browse a catalog, narrow it down with filters and search, inspect a single product, add items to a cart, and walk through a checkout that validates before it submits. There is no real payment processor here — the goal is the client-side machinery a store depends on. You will juggle server state (products fetched from an API) against client state (the cart), keep the cart alive across page reloads, and route between catalog, detail, and checkout views. It is the project where "just a bit of state" grows into something that needs a real strategy.

## Prerequisites

- Comfort building components and lifting state in a modern framework (React, Vue, Svelte, or Angular)
- Fetching data from a REST API and rendering async results
- Client-side routing basics (how a URL maps to a view)
- Familiarity with forms and controlled inputs
- Understanding of `localStorage` for persistence

## Learning Objectives

By the end, you should be able to:

- Separate server state (catalog) from client state (cart) and manage each appropriately
- Implement filtering, sorting, and search that compose without fighting each other
- Persist the cart across reloads and reconcile it with fresh product data
- Build a multi-field checkout form with per-field validation and clear error messaging
- Handle loading, empty, and error states for every data-driven view
- Make catalog and cart interactions keyboard- and screen-reader-accessible

## Functional Requirements

1. The catalog must fetch products from an API and render them in a responsive grid.
2. Users must be able to filter by category and price, search by keyword, and sort (price, rating, newest) — and these controls must combine.
3. Each product must have a detail view reachable by its own URL, so it can be shared and bookmarked.
4. Adding a product to the cart must update a visible cart badge; quantities must be adjustable and items removable.
5. The cart must survive a full page reload and recompute totals from current product prices.
6. Checkout must validate every required field before allowing submission and surface errors inline.
7. Every data view must show a loading state, an empty state, and a recoverable error state.

## Suggested Milestones

1. **Milestone 1 — Catalog:** Fetch and render the product grid with loading and error states.
2. **Milestone 2 — Filter, search, sort:** Layer composable controls over the catalog and reflect them in the URL.
3. **Milestone 3 — Detail & cart:** Add routed product pages and a persistent cart with quantity controls.
4. **Milestone 4 — Checkout:** Build the validated checkout form and an order confirmation screen.

## Data & Interface Sketch

```text
Layout
  [ Header: logo | search | cart badge(3) ]
  [ Sidebar filters ][ Product grid           ]
  Routes: /  /product/:id  /cart  /checkout  /confirmation

State
  products:  Product[]        (server state, fetched)
  cart:      CartItem[]       (client state, persisted)
  filters:   { q, category, minPrice, maxPrice, sort }  (mirrored in URL)

Product   { id, title, price, category, rating, image, stock }
CartItem  { productId, qty }   // store qty only; price read from product

API consumed
  GET /api/products?category=&q=&sort=  -> 200 Product[]
  GET /api/products/:id                 -> 200 Product | 404
```

## Stretch Goals

- Add a wishlist that persists separately from the cart.
- Show "related products" on the detail page from the same category.
- Debounce the search input and cancel stale in-flight requests.
- Add optimistic quantity updates that roll back on failure.
- Support a coupon code that adjusts the cart total.

## Definition of Done

- [ ] Filters, search, and sort combine correctly and are reflected in the URL.
- [ ] The cart persists across reloads and totals recompute from live prices.
- [ ] Checkout blocks submission until all required fields are valid, with inline errors.
- [ ] Loading, empty, and error states exist for the catalog and detail views.
- [ ] All interactive controls are reachable and operable by keyboard.

## Common Pitfalls

- Storing full product objects (including price) in the cart, so prices go stale after a catalog update.
- Treating filters as independent flags that overwrite each other instead of composing into one query.
- Forgetting the empty state — a blank grid after a filter looks like a bug.
- Validating the whole checkout form only on submit, leaving users to hunt for the one bad field.
- Losing focus when a modal cart drawer opens or closes.

## Resources

- [MDN: Using the Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) — data fetching fundamentals.
- [web.dev: Patterns](https://web.dev/patterns/) — reusable solutions for common UI problems including forms and loading.
- [MDN: Client-side form validation](https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation) — building trustworthy checkout validation.
- [MDN: Window.localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) — persisting the cart.
- [roadmap.sh: Frontend](https://roadmap.sh/frontend) — where state management and routing fit in the bigger picture.
