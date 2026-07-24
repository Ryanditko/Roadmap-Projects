# Dockerize a Simple App

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Take an application you already have — a small web server, an API, or a CLI — and package it so it runs identically on any machine with Docker. The point is not merely "it starts in a container" but a lean, reproducible image: a pinned base, only the files you actually need, a non-root user, and one cleanly exposed port. On the way you will meet the layer cache, the split between a build stage and a runtime stage, and why a `.dockerignore` earns its keep. When you are done, `docker run` hands anyone the exact environment you had — and retires the "works on my machine" excuse.

## Prerequisites

- An application that starts from a single command (any language)
- Docker installed locally with the daemon running
- Basic command-line comfort (paths, environment variables, ports)
- A clear idea of your app's runtime version and dependencies

## Learning Objectives

By the end, you should be able to:

- Write a Dockerfile that builds a runnable image from source
- Use a multi-stage build to keep build tooling out of the final image
- Explain the layer cache and order instructions to exploit it
- Inject configuration into a container via environment variables and ports
- Run the container as a non-root user and explain why that matters

## Functional Requirements

1. The image must build from a single `docker build` with no manual steps.
2. The running container must serve or execute the app on a documented port.
3. The build must be multi-stage so build tools are absent from the final image.
4. A `.dockerignore` must exclude source control, dependencies, and secrets from the build context.
5. The container must run as a non-root user.
6. The image must carry an explicit version tag, not only `latest`.
7. Configuration such as port and log level must be injectable via environment variables.

## Suggested Milestones

1. **Milestone 1 — First build:** Write a single-stage Dockerfile, build it, and run the app in a container.
2. **Milestone 2 — Slim it down:** Split into build and runtime stages, add `.dockerignore`, and pin the base image.
3. **Milestone 3 — Harden & tag:** Add a non-root user, environment-based config, a version tag, and a documented run command.

## Data & Interface Sketch

```text
Project layout
  Dockerfile
  .dockerignore
  <app source>

Dockerfile structure (stages, not the full file)
  stage "build":
    FROM <base>:<pinned-version>
    copy dependency manifest -> install deps   (cached layer)
    copy source -> compile/build artifact
  stage "runtime":
    FROM <slim-base>:<pinned-version>
    create + switch to non-root user
    copy artifact from "build"
    EXPOSE <port>
    ENV APP_PORT / LOG_LEVEL
    ENTRYPOINT / CMD

Commands
  docker build -t myapp:1.0.0 .
  docker run -p 8080:8080 -e LOG_LEVEL=info myapp:1.0.0
```

## Stretch Goals

- Push the image to a registry (Docker Hub, GHCR) and pull it on another machine.
- Add a `HEALTHCHECK` instruction so Docker reports container health.
- Scan the image with `docker scout` or Trivy and fix a real finding.
- Shrink further with an Alpine or distroless base and compare sizes.

## Definition of Done

- [ ] `docker build` produces an image with no errors from a clean checkout.
- [ ] The final image contains no compilers or build-only dependencies.
- [ ] The container runs as a non-root user (verify with `whoami` inside it).
- [ ] Config values change behavior via `-e` without a rebuild.
- [ ] The image carries an explicit semantic version tag.

## Common Pitfalls

- Copying the whole context (including `node_modules` or `.git`), inflating build time and image size.
- Placing `COPY . .` before dependency install, busting the cache on every source change.
- Running as root by default, leaving the container over-privileged.
- Baking secrets or environment-specific values into the image instead of injecting them at runtime.

## Resources

- [Docker: Dockerfile best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) — layer caching, multi-stage, and slimming.
- [Docker: Multi-stage builds](https://docs.docker.com/build/building/multi-stage/) — the official pattern for lean images.
- [Docker: .dockerignore reference](https://docs.docker.com/build/concepts/context/#dockerignore-files) — control what enters the build context.
- [Snyk: 10 Docker image security best practices](https://snyk.io/blog/10-docker-image-security-best-practices/) — non-root users and scanning.
