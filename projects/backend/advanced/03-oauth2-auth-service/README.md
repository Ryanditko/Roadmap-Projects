# High-Scale Authentication Service (OAuth2)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build the service every other service trusts: a centralized authorization server that issues and validates OAuth2 tokens for many client applications at once. You will implement the flows that power "Sign in with…" buttons and machine-to-machine access, then harden them for scale — where token validation happens millions of times a minute and a leaked signing key is a company-wide incident. The interesting work is not the happy path but the lifecycle: rotating keys without invalidating live sessions, revoking a stolen refresh token instantly, and letting resource servers validate access tokens without calling you on every request.

## Prerequisites

- Solid grasp of HTTP, TLS, and how cookies and `Authorization` headers travel
- Comfort with public-key cryptography (signing vs. encryption, key pairs)
- Experience building REST APIs with persistence and a cache layer
- Familiarity with JSON Web Tokens and their claims; read the OAuth2 spec before starting
- A backend stack of your choice (Node, Go, Java, Python) plus a datastore and Redis-like cache

## Learning Objectives

By the end, you should be able to:

- Implement the authorization code flow with PKCE and the client credentials flow
- Reason about access-token vs. refresh-token lifetimes and rotation
- Sign tokens with rotating keys exposed via a JWKS endpoint
- Design stateless validation so resource servers scale independently
- Revoke and introspect tokens, and audit every sensitive action
- Weigh the trade-off between self-contained JWTs and opaque reference tokens

## Functional Requirements

1. The service must implement the authorization code grant with PKCE and the client credentials grant per RFC 6749 and RFC 7636.
2. It must issue short-lived access tokens and longer-lived refresh tokens, with refresh-token rotation invalidating the prior token on use.
3. It must sign tokens with an asymmetric key and publish public keys at a JWKS endpoint so resource servers validate offline.
4. It must support scope-based authorization and reject requests for scopes a client is not registered for.
5. It must expose token introspection (RFC 7662) and revocation (RFC 7009) endpoints.
6. It must detect and reject refresh-token reuse, treating it as a compromise signal and revoking the token family.
7. **Availability:** validation must not depend on the auth server being reachable; JWKS is cached with a TTL.
8. **Throughput:** the token and introspection endpoints must sustain high request rates; hot paths (client lookup, key material) must be cached.
9. **Consistency:** revocation must propagate to resource servers within a bounded, documented staleness window.

## Suggested Milestones

1. **Milestone 1 — Authorization code + PKCE:** Register clients, run the full redirect flow, issue a signed access token.
2. **Milestone 2 — Refresh & rotation:** Add refresh tokens with rotation and reuse detection.
3. **Milestone 3 — Keys & validation:** Publish JWKS, rotate keys with overlap, validate tokens offline.
4. **Milestone 4 — Introspection, revocation & audit:** Add the management endpoints and an append-only audit log.

## Data & Interface Sketch

```text
Component view

  [Client App] --redirect--> [Auth Server] --consent--> [User]
        |  code + PKCE verifier        |
        v                              v
   POST /token  <----------------  [Client Store]
        |                              |
   access + refresh              [Signing Keys] --public--> GET /.well-known/jwks.json
        |
        v
  [Resource Server] --validate offline via cached JWKS--> allow/deny
        ^
        |-- introspect (opaque tokens) / check revocation list

Endpoints
  GET  /authorize        (response_type=code, code_challenge, scope, state)
  POST /token            (grant_type: authorization_code | refresh_token | client_credentials)
  POST /introspect       -> { active, scope, client_id, exp }
  POST /revoke           -> 200
  GET  /.well-known/jwks.json
```

## Stretch Goals

- Add the device authorization grant (RFC 8628) for TVs and CLI tools.
- Support OpenID Connect: add an `id_token` and a `/userinfo` endpoint.
- Add multi-factor authentication as a step-up during the authorize flow.
- Implement dynamic client registration and per-client rate limiting.

## Definition of Done

- [ ] A client completes the code + PKCE flow and receives a working, correctly-scoped access token.
- [ ] Access tokens validate offline against JWKS with no call to the auth server.
- [ ] Key rotation keeps existing tokens valid until they expire (overlapping key window).
- [ ] Reusing a rotated refresh token revokes the whole token family and is logged.
- [ ] Revocation and introspection endpoints behave per RFC, and every issue/revoke is audited.

## Common Pitfalls

- Long-lived access tokens: if they cannot be revoked, keep them short and lean on refresh rotation.
- Skipping `state` and PKCE, leaving the flow open to CSRF and authorization-code interception.
- Rotating signing keys with no overlap window, instantly invalidating every live token.
- Putting sensitive or fast-changing data in a JWT — you cannot un-issue it before expiry.
- Storing refresh tokens in plaintext instead of hashed, so a DB leak hands over live sessions.

## Resources

- [RFC 6749: The OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749) — the core spec.
- [RFC 7636: PKCE](https://datatracker.ietf.org/doc/html/rfc7636) — protecting public clients.
- [OAuth 2.0 Security Best Current Practice](https://datatracker.ietf.org/doc/html/rfc9700) — modern hardening guidance.
- [RFC 7519: JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519) — token structure and claims.
- [OWASP: JWT for Java Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html) — common validation mistakes.
