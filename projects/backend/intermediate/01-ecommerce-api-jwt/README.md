# E-commerce API with JWT Authentication

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build the backend for an online store: a catalog of products, a per-user shopping cart, and an order flow that turns a cart into a paid, inventory-adjusted order. Authentication is handled with JWTs, and access is gated by role — a shopper can browse and buy, an admin can manage the catalog. The interesting difficulty here is not any single endpoint but how they compose: reserving stock without overselling, keeping a checkout atomic, and issuing tokens that expire cleanly. This is where CRUD stops being enough and real transactional thinking begins.

## Prerequisites

- A REST API foundation ([Simple REST API for Task Management](../../beginner/01-simple-rest-api-task-management/) is a good base)
- Token-based auth basics ([Basic Authentication API](../../beginner/03-basic-authentication-api/) covers the groundwork)
- A relational or document database and its transaction model (PostgreSQL, MySQL, or MongoDB)
- Understanding of hashing passwords and signing/verifying JWTs

## Learning Objectives

By the end, you should be able to:

- Issue, verify, and expire JWT access tokens and design a refresh strategy
- Enforce role-based access control (RBAC) at the route and resource level
- Model products, carts, orders, and users with correct relationships
- Wrap checkout in a database transaction so stock and orders stay consistent
- Prevent overselling under concurrent purchases using locking or atomic updates

## Functional Requirements

1. Users must be able to register and log in, receiving a signed JWT on success.
2. Requests to protected endpoints must be rejected with 401 when the token is missing, expired, or invalid.
3. Admin-only endpoints (create/update/delete product) must return 403 for non-admin tokens.
4. A user must be able to add products to a cart, update quantities, and remove items.
5. Checkout must create an order, decrement inventory, and empty the cart as one atomic operation.
6. An order must never be created for a quantity greater than the available stock.
7. Users must be able to list their own past orders and view a single order's detail.
8. Product listing must support filtering by category and pagination.

## Suggested Milestones

1. **Milestone 1 — Auth & roles:** Registration, login, JWT issuance, and middleware that verifies tokens and enforces roles.
2. **Milestone 2 — Catalog & cart:** Product CRUD (admin) and cart operations scoped to the authenticated user.
3. **Milestone 3 — Checkout & orders:** Transactional order creation with inventory decrement and order history.
4. **Milestone 4 — Concurrency hardening:** Reproduce an oversell under parallel requests, then fix it with locking or atomic decrements.

## Data & Interface Sketch

```text
User    id, email, passwordHash, role ("customer" | "admin")
Product id, name, priceCents, stock, categoryId
Cart    userId, items: [{ productId, qty }]
Order   id, userId, status, totalCents, lines: [{ productId, qty, unitPriceCents }]

POST /auth/register  -> 201 { id, email }
POST /auth/login     -> 200 { accessToken, refreshToken }
GET  /products?category=&page=  -> 200 { items, page, total }
POST /products       (admin)    -> 201 { id, ... }   | 403
POST /cart/items     body: { productId, qty }        -> 200 cart
POST /orders/checkout            -> 201 order | 409 (insufficient stock)
GET  /orders         -> 200 [ ...own orders ]

Authorization: Bearer <jwt>   (claims: sub, role, exp)
```

## Stretch Goals

- Add refresh-token rotation with server-side revocation on logout.
- Support promotional/discount codes applied at checkout with validation.
- Add product reviews and an aggregate rating per product.
- Emit an order-confirmation event (mock email or webhook) after checkout.

## Definition of Done

- [ ] Tokens expire and expired tokens are rejected with 401, not silently accepted.
- [ ] Non-admin callers cannot create, edit, or delete products (verified with a test).
- [ ] Checkout is atomic: a failure mid-way leaves stock and the cart unchanged.
- [ ] Concurrent checkouts for the last unit never both succeed.
- [ ] A user can only see their own orders, never another user's.

## Common Pitfalls

- Trusting the JWT payload without verifying the signature — anyone can forge claims otherwise.
- Checking stock and decrementing it in two separate queries, leaving a race window that oversells.
- Storing prices as floats; use integer cents to avoid rounding drift on totals.
- Putting role checks only in the client or in some routes but not others.
- Making checkout a series of independent writes with no transaction, so a crash leaves a half-order.

## Resources

- [RFC 7519: JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519) — the JWT specification.
- [OWASP: Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) — how to enforce RBAC correctly.
- [PostgreSQL: Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html) — the semantics you rely on for atomic checkout.
- [roadmap.sh: Backend Developer](https://roadmap.sh/backend) — where auth and databases sit in the bigger picture.
