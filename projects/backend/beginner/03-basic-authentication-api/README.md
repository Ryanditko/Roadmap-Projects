# Basic Authentication API

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Beginner · **Estimated time:** 5–8 hours

## Overview

Build the login layer that sits in front of a protected API: a user submits credentials, the server verifies them and issues a token, and subsequent requests must carry that token to reach protected routes. You start from a small set of predefined users so you can focus on the flow itself — authentication versus authorization, token issuance, and gatekeeping middleware — rather than user registration or a database. Every real backend has this layer; here you build it from first principles.

## Prerequisites

- Solid grasp of building HTTP endpoints ([Simple REST API for Task Management](../01-simple-rest-api-task-management/) recommended first)
- Understanding of HTTP headers, especially `Authorization`
- A web framework that supports middleware/hooks
- Awareness that credentials must never be logged or hardcoded in plaintext

## Learning Objectives

By the end, you should be able to:

- Distinguish authentication (who you are) from authorization (what you may do)
- Verify a password against a stored hash instead of comparing plaintext
- Issue a token on login and validate it on protected requests
- Implement middleware that gates routes based on token presence and validity
- Reason about token expiration and why sessions should not live forever
- Return the right status codes for auth failures: 401 versus 403

## Functional Requirements

1. The system must expose a login endpoint that accepts a username and password.
2. The system must verify the password against a stored hash, never a plaintext comparison.
3. On success, the system must return a token; on failure, it must return 401 with no detail about which field was wrong.
4. Protected endpoints must reject requests lacking a valid token with 401.
5. The system must expose at least one protected endpoint that returns the caller's identity.
6. Tokens must carry or reference an expiration, after which they are rejected.
7. The system must support logout or token invalidation.

## Suggested Milestones

1. **Milestone 1 — Verify credentials:** Store users with hashed passwords and return a token on a correct login.
2. **Milestone 2 — Guard routes:** Add middleware that reads the `Authorization` header and blocks invalid tokens.
3. **Milestone 3 — Lifecycle & roles:** Add expiration, logout/invalidation, and optionally a role check (admin vs user).

## Data & Interface Sketch

```text
User (seed data)
  username:     string
  passwordHash: string   (never plaintext)
  role:         "admin" | "user"

POST /login              body: { username, password }
                         -> 200 { token, expiresAt } | 401
GET  /me                 header: Authorization: Bearer <token>
                         -> 200 { username, role } | 401
POST /logout             -> 204   (invalidate token)

Protected request rule:
  missing/expired/invalid token -> 401
  valid token but insufficient role -> 403
```

## Stretch Goals

- Replace opaque tokens with signed JWTs and verify the signature on each request.
- Add role-based authorization so admin-only routes reject regular users with 403.
- Add a refresh-token flow to renew access without re-entering credentials.
- Rate-limit failed logins to slow down brute-force attempts.

## Definition of Done

- [ ] Login succeeds only with correct credentials and returns a token.
- [ ] Passwords are stored and compared as hashes, verified by reading the code.
- [ ] Protected routes return 401 for missing, malformed, or expired tokens.
- [ ] Wrong password and unknown user both return the same generic 401.
- [ ] Logout (or expiry) makes a previously valid token stop working.

## Common Pitfalls

- Comparing passwords with `==` instead of a hash-verify function — this leaks and is insecure.
- Telling the client whether the username or the password was wrong, which helps attackers enumerate users.
- Confusing 401 (not authenticated) with 403 (authenticated but not allowed).
- Signing tokens with a hardcoded secret committed to the repo; read it from configuration instead.

## Resources

- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) — the authoritative do's and don'ts.
- [MDN: HTTP Authorization header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) — how the token travels.
- [jwt.io Introduction](https://jwt.io/introduction) — what a JSON Web Token is and how it is structured.
- [MDN: 401 vs 403](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401) — when to use each auth status code.
