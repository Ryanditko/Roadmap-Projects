# File Ingestion System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Many pipelines start with a "drop folder": another team, a partner, or an export job leaves files in a directory, and your system must notice them, process them, and move them out of the way. Build a service that watches an incoming directory, picks up each new file, validates and processes it, and then moves it to a "done" or "failed" folder so it is never processed twice. The subtle challenges — a file that is still being written when you grab it, two runs racing over the same file, a corrupt file that must not block the queue — are exactly what makes ingestion a real engineering problem.

## Prerequisites

- Comfort listing directories and moving/renaming files
- Understanding of file paths and basic filesystem operations
- Beginner error handling and logging
- Awareness that a file can be partially written (not yet complete)

## Learning Objectives

By the end, you should be able to:

- Detect new files in a watched directory reliably
- Avoid grabbing a file that is still being written (stability check)
- Process each file exactly once and move it to a terminal folder
- Route failed files to a quarantine folder instead of retrying forever
- Make the pickup safe against a second run racing on the same file
- Log every ingestion event so the pipeline is auditable

## Functional Requirements

1. The system must scan an `incoming/` directory and detect files that have not yet been processed.
2. It must confirm a file is complete before processing (e.g. size stable across two checks, or an atomic rename convention).
3. Each file must be validated (type/extension and basic structure) before it is processed.
4. On success, the file must move to `processed/`; on failure, to `failed/` with an error record.
5. A file must never be processed twice, even across restarts or overlapping runs.
6. Every action (detected, processed, failed) must be logged with a timestamp and the filename.

## Suggested Milestones

1. **Milestone 1 — Detect & move:** Scan `incoming/`, process each file, and move it to `processed/`.
2. **Milestone 2 — Safety:** Add the completeness check and a claim mechanism so the same file is not double-processed.
3. **Milestone 3 — Failure handling:** Validate files, quarantine bad ones in `failed/`, and log every event.

## Data & Interface Sketch

```text
directory layout
  incoming/   <- files arrive here
  processing/ <- claimed files (in-flight)
  processed/  <- success
  failed/     <- quarantine + <name>.error.txt

per-file flow
  detect incoming/orders_2024-05-01.csv
  completeness check (size stable? or .ready marker?)
  claim: atomic rename -> processing/  (wins the race)
  validate (extension=csv, header present)
  process -> [ok] move to processed/
          \-> [bad] move to failed/ + write reason

ingest.log
  2024-05-01T09:00 detected orders_...csv
  2024-05-01T09:00 processed orders_...csv rows=812
```

## Stretch Goals

- Add a continuous watch loop (polling interval or OS filesystem events) instead of a one-shot scan.
- Support multiple file types with a handler chosen by extension.
- Add a retry policy with a maximum attempt count before quarantine.
- Emit a daily summary of files ingested, rows processed, and failures.

## Definition of Done

- [ ] New files in `incoming/` are detected and processed.
- [ ] A file still being written is not picked up until complete.
- [ ] Successful files land in `processed/`, failures in `failed/` with a reason.
- [ ] No file is processed twice, even with two runs overlapping.
- [ ] Every ingestion event is logged with timestamp and filename.

## Common Pitfalls

- Grabbing a file mid-write and processing a truncated, corrupt copy.
- Two runs both claiming the same file because the claim is not atomic.
- Leaving failed files in `incoming/`, so they are retried forever and block the queue.
- Relying on modification time alone to detect "new" files, which is fragile across clocks.

## Resources

- [Python `pathlib`](https://docs.python.org/3/library/pathlib.html) — clean, cross-platform filesystem operations.
- [watchdog library](https://python-watchdog.readthedocs.io/en/stable/) — OS-level filesystem event notifications.
- [POSIX `rename` atomicity](https://pubs.opengroup.org/onlinepubs/9699919799/functions/rename.html) — why an atomic rename is the safe claim primitive.
- [The Log: What every engineer should know](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying) — ingestion as the front of a data system.
