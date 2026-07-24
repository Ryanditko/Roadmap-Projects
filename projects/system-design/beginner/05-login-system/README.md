# Design an Authentication & Login System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Design the authentication system behind a web application: how users register, log in, stay logged in across requests, and log out. The subtle part is that HTTP is stateless, so "staying logged in" requires either a server-side session or a signed token. You'll reason about how passwords are stored safely, how a session or token proves identity on each request, and how to defend against brute-force and stolen credentials. Deliver a design document covering the auth flow, the token/session model, and the security decisions — no working auth code.

## Prerequisites

- Understanding that HTTP is stateless and each request stands alone
- Awareness that passwords must never be stored in plain text
- Familiarity with cookies and HTTP headers at a basic level
- Comfort reasoning about the difference between authentication and authorization

## Learning Objectives

By the end, you should be able to:

- Design a registration and login flow with safe password storage
- Compare server-side sessions with stateless tokens (JWT) and justify a choice
- Reason about token expiry, refresh, and revocation
- Design defenses against brute-force attacks
- State a trade-off between stateless scalability and revocation control

## Requirements & Constraints

1. Users register with an email (e.g., `name@example.com`) and password, then log in.
2. Passwords are stored using a slow, salted hash — never reversible.
3. An authenticated session persists across requests until it expires or the user logs out.
4. The system resists brute-force login attempts.
5. Logout and token/session revocation must actually invalidate access.
6. Estimate scale: 2M users, 500K logins/day — roughly 6 logins/s average.

## Suggested Approach

1. Separate the one-time login exchange from the per-request proof of identity.
2. Design password storage: a slow hash (bcrypt/argon2) with a per-user salt.
3. Choose the session mechanism — server-side session store vs. signed stateless token — and note how each is validated per request.
4. Add rate limiting and lockout on repeated failures.
5. Describe how logout works in your chosen model, including token revocation.

## Architecture Sketch

```text
Client ── POST /register ─> [ Auth Service ] ─> [ User Store (email, pw_hash, salt) ]
Client ── POST /login ────> [ Auth Service ] ─> verify hash -> issue token/session
Client ── GET /me ────────> [ App ] -> validate token/session -> [ Session Store? ]
Client ── POST /logout ───> [ Auth Service ] -> revoke session / blocklist token

Core API
  POST /register  { email, password }   -> 201 { userId }
  POST /login     { email, password }    -> 200 { token }  (+ Set-Cookie)
  GET  /me        (Authorization)         -> 200 { userId } | 401
  POST /logout    (Authorization)         -> 204

Data model
  users:    user_id (PK) | email | password_hash | salt | created_at
  sessions: session_id (PK) | user_id | expires_at   (if session-based)
  attempts: email/ip | count | window_start          (for rate limiting)
```

## Deep-Dive Topics

- **Password storage:** why slow hashes (bcrypt, argon2) beat fast ones, and the role of a per-user salt.
- **Sessions vs. tokens:** the revocation problem with stateless JWTs and mitigations (short expiry + refresh, blocklist).
- **Brute-force defense:** rate limiting, exponential lockout, and CAPTCHA as escalation.

## Deliverables

- An architecture diagram showing register, login, authenticated request, and logout.
- The core API contract for the auth endpoints.
- A data model for users and sessions/tokens.
- One trade-off written up: e.g., server-side sessions (easy revocation, but a shared session store to scale) vs. stateless JWTs (scale freely, but revocation is hard before expiry).

## Common Pitfalls

- Storing passwords with a fast hash (MD5/SHA-256) or, worse, in plain text.
- Using long-lived stateless tokens with no revocation path, so a stolen token stays valid.
- Skipping rate limiting, leaving login open to credential stuffing.
- Confusing authentication (who you are) with authorization (what you may do).

## Resources

- [OWASP: Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) — practical, authoritative guidance.
- [OWASP: Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) — how to hash passwords.
- [RFC 7519: JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519) — the token standard.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — sessions, security, and scaling patterns.
