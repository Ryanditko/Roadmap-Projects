# Environment Config Manager

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build a small tool that loads the right configuration for the right environment — `dev`, `staging`, `prod` — and hands the application one validated set of values. The core idea is layering: a shared base config, overridden by environment-specific files, overridden by environment variables at the top. Along the way you separate secrets from plain settings, verify that required keys exist before the app starts, and fail loudly on a bad config instead of crashing mysteriously three requests later. This is the humble tool that makes "twelve-factor" config real: one codebase, many deploys, no `if (env === 'prod')` scattered through the source.

## Prerequisites

- A language with a config or file-parsing library (any stack)
- Understanding of environment variables and how processes read them
- A file format to work in: JSON, YAML, or TOML
- Awareness that secrets must never be committed to source control

## Learning Objectives

By the end, you should be able to:

- Layer configuration sources with a clear precedence order
- Separate non-secret settings from secrets and load each appropriately
- Validate configuration against a schema before the app uses it
- Provide sensible defaults while allowing per-environment overrides
- Fail fast with a clear message when required config is missing

## Functional Requirements

1. The tool must load a base config and merge environment-specific overrides on top of it.
2. Environment variables must take precedence over file values for the same key.
3. Required keys must be validated on load; a missing required key must abort startup with a clear error.
4. It must support at least three environments selected by a single variable (e.g. `APP_ENV`).
5. Secrets must be sourced separately (from the environment or a git-ignored secrets file).
6. The resolved config must be exposed to the app as one structured object.
7. It must never print secret values in logs or error messages.

## Suggested Milestones

1. **Milestone 1 — Load & merge:** Read a base file and an environment file, merging them with defined precedence.
2. **Milestone 2 — Env overrides & secrets:** Let environment variables override file values and load secrets separately.
3. **Milestone 3 — Validate & fail fast:** Add a required-keys schema and abort with a helpful message on a bad config.

## Data & Interface Sketch

```text
Precedence (lowest -> highest)
  defaults  <  config.base.yml  <  config.<env>.yml  <  environment variables

File layout
  config/
    base.yml
    development.yml
    production.yml
  .env            (git-ignored, secrets)
  .gitignore      (excludes .env)

Config schema (key names + types, not real values)
  app:
    port:        integer   (required)
    log_level:   enum[debug|info|warn|error]  (default info)
  database:
    host:        string    (required)
    password:    secret     (required, from env only)
  feature_flags: map<string, boolean>

Resolution
  APP_ENV=production -> merge base + production + env vars -> validate -> Config object
```

## Stretch Goals

- Add a `config validate` command that checks a config without starting the app.
- Print a redacted dump of the resolved config (secrets shown as `***`).
- Support hot-reload: detect a changed file and re-validate without a restart.
- Generate a `.env.example` listing every required key with placeholder values.

## Definition of Done

- [ ] Switching `APP_ENV` loads a different, correctly-merged configuration.
- [ ] An environment variable overrides the same key set in a file.
- [ ] Removing a required key aborts startup with a message naming the missing key.
- [ ] Secrets are not committed and never appear in logs or the redacted dump.
- [ ] The application receives one structured config object, not scattered lookups.

## Common Pitfalls

- Committing a `.env` or secrets file — add it to `.gitignore` before the first commit.
- Getting precedence backwards, so a base default silently overrides an environment variable.
- Reading config ad hoc throughout the code instead of resolving once at startup.
- Logging the whole config object on boot and leaking a password into the logs.

## Resources

- [The Twelve-Factor App: Config](https://12factor.net/config) — why config belongs in the environment.
- [OWASP: Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html) — handling secrets safely.
- [TOML specification](https://toml.io/en/) — a config format designed to be unambiguous.
- [YAML specification](https://yaml.org/spec/) — the format and its footguns.
