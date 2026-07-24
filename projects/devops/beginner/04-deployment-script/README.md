# Simple Deployment Script

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Write a script that takes a new version of your app and puts it on a server — reliably, repeatably, and without a checklist you have to remember at midnight. The script connects to the host, places the new code, restarts the service, and checks that it actually came back up. The interesting part is not the happy path but the guarantees around it: take a backup before you touch anything, verify health afterward, and roll back to the previous version if the new one refuses to start. When you finish, deploying should be one command that either succeeds cleanly or leaves the server exactly as it was.

## Prerequisites

- An app that runs on a remote (or virtual) Linux host you can SSH into
- SSH key-based access to that host
- A service manager to (re)start the app (systemd, Docker, or a process manager)
- Basic shell scripting (variables, conditionals, exit codes)

## Learning Objectives

By the end, you should be able to:

- Automate a remote deploy over SSH from a single command
- Snapshot the current release so you can restore it
- Restart a service and confirm it is healthy before declaring success
- Roll back automatically when a deploy fails its health check
- Make the script idempotent and safe to re-run

## Functional Requirements

1. The script must connect to a target host and transfer the new release non-interactively.
2. It must back up the current release before replacing it.
3. It must restart the service after placing the new code.
4. It must run a health check and treat a failed check as a failed deploy.
5. On a failed health check, it must roll back to the backed-up release and restart.
6. It must exit non-zero on any failure so a caller (or CI) can detect it.
7. It must log each step with a timestamp for later diagnosis.

## Suggested Milestones

1. **Milestone 1 — Push & restart:** Copy the release to the host and restart the service over SSH.
2. **Milestone 2 — Backup & verify:** Snapshot the current release first, then health-check after restart.
3. **Milestone 3 — Rollback & logging:** Restore the previous release on failure and log every step with exit codes.

## Data & Interface Sketch

```text
Usage
  deploy.sh <environment> <release-ref>

Config (env vars, not hardcoded)
  DEPLOY_HOST      user@host
  DEPLOY_PATH      /srv/app
  HEALTH_URL       http://localhost:8080/health
  SERVICE_NAME     app.service

Steps (structure, not full script)
  1. resolve config + validate inputs
  2. ssh: snapshot current release -> releases/<timestamp>
  3. transfer new release -> DEPLOY_PATH
  4. ssh: restart SERVICE_NAME
  5. poll HEALTH_URL (retry N times, backoff)
     ok    -> log success, exit 0
     fail  -> restore snapshot, restart, log failure, exit 1
```

## Stretch Goals

- Keep the last N releases and support a `rollback` subcommand to any of them.
- Use a symlink-swap strategy (`current -> releases/<ts>`) for near-zero-downtime cutover.
- Send a notification (e.g. to a chat webhook) on success and failure.
- Add a `--dry-run` flag that prints each action without executing it.

## Definition of Done

- [ ] A full deploy runs from one command with no interactive prompts.
- [ ] A backup of the previous release exists on the host after deploy.
- [ ] A deliberately broken release triggers an automatic rollback.
- [ ] The service is confirmed healthy before the script reports success.
- [ ] The script exits non-zero on failure and logs every step.

## Common Pitfalls

- Assuming the restart succeeded without checking — a service can "start" and immediately crash.
- Overwriting the running release before backing it up, leaving nothing to roll back to.
- Hardcoding host, path, or credentials into the script instead of reading them from config.
- Health-checking once with no retries, so a slow-starting app is misread as broken.

## Resources

- [DigitalOcean: How to use SSH keys](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server) — non-interactive remote access.
- [Google SRE Book: Release Engineering](https://sre.google/sre-book/release-engineering/) — the principles behind safe, repeatable deploys.
- [rsync manual](https://download.samba.org/pub/rsync/rsync.1) — efficient, resumable file transfer for releases.
- [systemd: Managing services](https://www.freedesktop.org/software/systemd/man/systemctl.html) — starting, restarting, and checking service state.
