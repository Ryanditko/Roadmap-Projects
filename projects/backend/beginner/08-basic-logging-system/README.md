# Basic Logging System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Beginner · **Estimated time:** 4–7 hours

## Overview

Build a small logging library that records application events to files with severity levels, timestamps, and file rotation — the thing every production service relies on but few beginners build from scratch. By constructing your own logger you learn what frameworks like Winston, Bunyan, or Python's `logging` actually do: filter by level, format consistently, write safely, and stop a log file from growing forever. You will then wire it into a sample app to see it work in context.

## Prerequisites

- Comfort with your language's file I/O
- Understanding of the standard severity levels (DEBUG, INFO, WARN, ERROR)
- Familiarity with timestamps and ISO 8601 formatting
- A small existing app (any earlier project) to instrument

## Learning Objectives

By the end, you should be able to:

- Model log severity levels and filter output by a configurable threshold
- Produce consistently formatted log lines with timestamp, level, and message
- Write logs to a file safely, appending without losing existing content
- Rotate log files by size or date so they never grow unbounded
- Compare human-readable and structured (JSON) log formats and when each fits

## Functional Requirements

1. The system must support at least four severity levels with a defined order.
2. The system must suppress messages below a configurable minimum level.
3. Every log line must include an ISO 8601 timestamp, the level, and the message.
4. Logs must be appended to a file, preserving earlier entries across restarts.
5. The system must rotate the log file when it exceeds a size threshold (or on a date boundary).
6. Rotated files must be retained up to a configurable count, with the oldest removed.
7. The minimum level and output destination must be configurable without code changes.

## Suggested Milestones

1. **Milestone 1 — Leveled logging:** Emit formatted, timestamped lines and honor a minimum-level filter.
2. **Milestone 2 — File output:** Append logs to a file that survives restarts, then instrument a sample app.
3. **Milestone 3 — Rotation:** Rotate by size, keep N old files, delete the rest; optionally add a JSON format.

## Data & Interface Sketch

```text
Level order: DEBUG < INFO < WARN < ERROR

Text line format:
  2026-07-24T14:03:22Z  INFO   user login succeeded  {userId=42}

JSON line format:
  { "ts": "...", "level": "INFO", "msg": "...", "ctx": { "userId": 42 } }

Rotation:
  app.log grows past MAX_BYTES
    -> rename app.log -> app.log.1  (shift .1 -> .2, ...)
    -> keep at most KEEP files, delete older
```

## Stretch Goals

- Add a JSON output mode toggled by configuration, alongside the text format.
- Support multiple simultaneous destinations (console + file) with independent levels.
- Add contextual fields (request id, user id) carried through a log call.
- Add time-based rotation (daily) in addition to size-based.

## Definition of Done

- [ ] Messages below the configured level are not written.
- [ ] Every line has a valid ISO 8601 timestamp and its level.
- [ ] Logs persist and append correctly across restarts.
- [ ] The file rotates at the size threshold and old files are pruned to the configured count.
- [ ] Level and destination are set via configuration, not hardcoded.

## Common Pitfalls

- Opening the file in truncate mode instead of append, wiping history on every start.
- Formatting timestamps in local time without an offset, making logs ambiguous across machines.
- Rotating by renaming while a write is in flight, losing or interleaving lines — coordinate the swap.
- Comparing levels as strings instead of ranked values, so filtering behaves unpredictably.

## Resources

- [Python logging HOWTO](https://docs.python.org/3/howto/logging.html) — the canonical model for levels and handlers.
- [The Twelve-Factor App: Logs](https://12factor.net/logs) — treating logs as event streams.
- [RFC 5424: Syslog severity levels](https://datatracker.ietf.org/doc/html/rfc5424#section-6.2.1) — the origin of standard levels.
- [Wikipedia: Log rotation](https://en.wikipedia.org/wiki/Log_rotation) — the rotation strategies you are implementing.
