# URL Shortener (In-Memory)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Beginner · **Estimated time:** 4–7 hours

## Overview

Build a service like Bitly or TinyURL that turns a long link into a short code and redirects visitors back to the original when they hit it. Everything lives in an in-memory map, so it resets on restart — which keeps the focus on the interesting parts: generating collision-free short codes and issuing correct HTTP redirects. It is a small project that quietly teaches encoding, routing, and the difference between a 301 and a 302.

## Prerequisites

- Comfort building basic HTTP endpoints ([Simple REST API for Task Management](../01-simple-rest-api-task-management/) is a good warm-up)
- Understanding of what a URL is made of (scheme, host, path)
- A web framework of your choice (Node, Python, or Go)
- Familiarity with maps/dictionaries as key-value stores

## Learning Objectives

By the end, you should be able to:

- Generate short, unique codes and reason about collision probability
- Implement HTTP redirects and choose between 301 (permanent) and 302 (temporary)
- Validate and normalize user-supplied URLs before storing them
- Use a key-value structure for bidirectional lookup (code ↔ URL)
- Track and expose simple per-link usage analytics

## Functional Requirements

1. The system must accept a long URL and return a unique short code.
2. The system must reject input that is not a syntactically valid `http`/`https` URL with a 400.
3. Visiting a short code must redirect the client to the original URL with an appropriate 3xx status.
4. Requesting an unknown code must return 404.
5. The system must guarantee that two different long URLs never receive the same code.
6. The system must count how many times each short code has been visited.
7. The system must expose an endpoint to retrieve the original URL and hit count without redirecting.

## Suggested Milestones

1. **Milestone 1 — Shorten & store:** Accept a URL, generate a code, store the mapping, return the short link.
2. **Milestone 2 — Redirect:** Resolve a code to its URL and issue the redirect, handling the 404 case.
3. **Milestone 3 — Validation & analytics:** Validate URLs on input and track visit counts per code.

## Data & Interface Sketch

```text
Link
  code:      string   (e.g. "aZ3kQ")
  longUrl:   string
  hits:      integer
  createdAt: ISO-8601 string

POST /shorten        body: { "url": "https://..." }
                     -> 201 { "code": "aZ3kQ", "shortUrl": ".../aZ3kQ" }
GET  /{code}         -> 302 Location: <longUrl>  | 404
GET  /api/{code}     -> 200 { longUrl, hits, createdAt } | 404

Code generation options: random base62, incrementing counter -> base62
```

## Stretch Goals

- Allow a user-chosen custom alias, rejecting one that is already taken.
- Add optional expiration so a code stops resolving after a set time.
- Return the shortened link for a URL that was already shortened, instead of a new code.
- Add a basic stats page listing top links by hits.

## Definition of Done

- [ ] A shortened link redirects to exactly the original URL, query string included.
- [ ] Invalid or non-HTTP URLs are rejected with 400 before any code is generated.
- [ ] Unknown codes return 404, not a redirect to nowhere.
- [ ] Generated codes are URL-safe and verified unique against the store.
- [ ] Hit counts increment on redirect and are visible via the stats endpoint.

## Common Pitfalls

- Storing the URL without validating it, then redirecting to `javascript:` or a malformed target.
- Using a hash of the URL as the code without handling the (rare) collision — decide on a strategy and test it.
- Confusing 301 and 302: browsers cache 301 aggressively, so a wrong permanent redirect is hard to undo.
- Forgetting to preserve the query string or fragment of the original URL on redirect.

## Resources

- [MDN: Redirections in HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections) — 301 vs 302 vs 307/308 explained.
- [RFC 3986: URI Generic Syntax](https://datatracker.ietf.org/doc/html/rfc3986) — the authority on what a valid URL looks like.
- [Wikipedia: Base62](https://en.wikipedia.org/wiki/Base62) — the common encoding for short, readable codes.
- [MDN: URL API](https://developer.mozilla.org/en-US/docs/Web/API/URL) — a built-in way to parse and validate URLs.
