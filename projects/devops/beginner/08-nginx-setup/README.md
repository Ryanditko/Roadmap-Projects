# Basic Nginx Setup

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Stand up Nginx and put it in front of an application, first as a plain web server and then as a reverse proxy that forwards requests to your app running behind it. This is one of the most common shapes in production: Nginx terminates TLS, serves static files fast, and hands dynamic requests to an upstream. You will write a server block, enable HTTPS, add a few sensible headers, and prove that a request to port 443 reaches your app and comes back correctly. The goal is a config you understand line by line — not a copied snippet you are afraid to touch.

## Prerequisites

- A running application listening on a local port (any language)
- A machine (or VM/container) where you can install and run Nginx
- Basic understanding of HTTP, ports, and DNS/hostnames
- A TLS certificate (self-signed for local, or Let's Encrypt for a real domain)

## Learning Objectives

By the end, you should be able to:

- Configure a server block that serves a site on a hostname and port
- Set up Nginx as a reverse proxy to an upstream application
- Enable HTTPS and redirect HTTP to HTTPS
- Add security and caching headers and gzip compression
- Read the access and error logs to debug a misbehaving config

## Functional Requirements

1. Nginx must serve a response for a defined hostname on port 80.
2. It must reverse-proxy requests to an upstream app and return the app's response.
3. HTTPS must be enabled on port 443 with a valid (or self-signed) certificate.
4. Plain HTTP requests must redirect to HTTPS.
5. The config must set forwarded headers (host, real IP, protocol) for the upstream.
6. Static assets must be served directly by Nginx with a cache header.
7. `nginx -t` must pass and a reload must apply changes without dropping connections.

## Suggested Milestones

1. **Milestone 1 — Serve:** Write a server block that returns a page for your hostname over HTTP.
2. **Milestone 2 — Proxy & TLS:** Reverse-proxy to the app, enable HTTPS, and redirect HTTP to HTTPS.
3. **Milestone 3 — Harden & tune:** Add security headers, gzip, static caching, and validate with `nginx -t`.

## Data & Interface Sketch

```text
File layout
  /etc/nginx/nginx.conf
  /etc/nginx/sites-available/app.conf   -> symlinked into sites-enabled/

Server block structure (key directives, not full config)
  server (port 80):
    server_name <host>
    return 301 https://$host$request_uri     # redirect to HTTPS
  server (port 443, ssl):
    server_name <host>
    ssl_certificate / ssl_certificate_key
    location /static/  -> root <path>; cache-control header
    location /         -> proxy_pass http://upstream_app
                          proxy_set_header Host / X-Real-IP / X-Forwarded-Proto
  upstream upstream_app:
    server 127.0.0.1:8080

Validate & apply
  nginx -t          # test config
  nginx -s reload   # graceful reload
```

## Stretch Goals

- Add a second upstream server and balance across them (round-robin).
- Add basic rate limiting to a login or API path.
- Automate certificate issuance and renewal with Certbot / Let's Encrypt.
- Add a `/healthz` location that returns 200 without hitting the upstream.

## Definition of Done

- [ ] A browser reaching the hostname over HTTPS gets the app's response.
- [ ] HTTP requests redirect to HTTPS.
- [ ] Static files are served by Nginx with a cache header, not proxied.
- [ ] Forwarded headers reach the upstream (verify the app sees the real client protocol/IP).
- [ ] `nginx -t` passes and a reload applies changes with no dropped requests.

## Common Pitfalls

- Forgetting `proxy_set_header Host $host`, so the upstream sees the wrong host and generates broken links.
- Leaving a self-signed cert in production and training users to click through TLS warnings.
- Editing `nginx.conf` and reloading without running `nginx -t` first, taking the site down on a typo.
- Serving static files through the proxy, wasting the upstream on work Nginx does faster.

## Resources

- [Nginx: Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html) — the official starting point.
- [Nginx: Reverse proxy guide](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/) — proxy_pass and forwarded headers.
- [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/) — safe, current TLS settings for Nginx.
- [Let's Encrypt / Certbot](https://certbot.eff.org/) — free certificates and automatic renewal.
