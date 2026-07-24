# Cron Job Manager

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Raw `cron` runs your jobs but tells you almost nothing: no history, no alert when a job fails, no protection against a slow run overlapping the next. Build a thin manager that wraps scheduled jobs and adds the operational layer they are missing. It reads a list of jobs and their schedules, runs each at the right time, captures output and exit code, prevents a job from running while a previous instance is still going, and alerts when one fails or overruns. When you finish, you will understand what cron actually guarantees — and what you have to add yourself to trust it in production.

## Prerequisites

- A few commands or scripts you want to run on a schedule
- A scripting language for the manager (Python, Go, or shell)
- Understanding of processes, exit codes, and standard output/error
- Familiarity with cron expression syntax

## Learning Objectives

By the end, you should be able to:

- Parse cron-style schedules and determine when each job is due
- Run a job as a subprocess and capture its output and exit code
- Prevent overlapping runs of the same job with a lock
- Record a run history and alert on failure or overrun
- Enforce a per-job timeout so a stuck job cannot run forever

## Functional Requirements

1. The manager must read a list of jobs, each with a command and a cron schedule.
2. It must run each job at its scheduled time and capture stdout, stderr, and exit code.
3. It must prevent a job from starting if its previous run is still in progress (locking).
4. It must record a history entry per run: start, end, exit code, and status.
5. It must alert when a job exits non-zero or exceeds its configured timeout.
6. It must terminate a job that exceeds its timeout.
7. Jobs, schedules, timeouts, and alert settings must be configurable.

## Suggested Milestones

1. **Milestone 1 — Schedule & run:** Parse schedules, run jobs on time, and capture output and exit code.
2. **Milestone 2 — Lock & history:** Add per-job locking against overlap and persist a run history.
3. **Milestone 3 — Timeout & alert:** Enforce timeouts, kill overruns, and alert on failure or overrun.

## Data & Interface Sketch

```text
Job config (structure, not full file)
  jobs:
    - name: nightly-report
      schedule: "0 3 * * *"     # cron expression
      command: ["/usr/bin/report", "--daily"]
      timeout_seconds: 600
      on_failure: alert
    - name: cache-warm
      schedule: "*/15 * * * *"
      command: ["./warm-cache.sh"]
      allow_overlap: false

Run record (persisted per execution)
  { job, run_id, started_at, ended_at, exit_code, status, output_ref }
  status: success | failed | timed_out | skipped_locked

CLI
  manager run              # daemon: evaluate schedules, dispatch due jobs
  manager list             # jobs + next run time
  manager history <job>    # recent runs and outcomes
```

## Stretch Goals

- Add job dependencies: run B only after A succeeds.
- Add a retry policy with backoff for transient failures.
- Expose a small status page or endpoint with last-run outcomes.
- Support environment variables and a working directory per job.

## Definition of Done

- [ ] Jobs run at their scheduled times and their exit codes are recorded.
- [ ] A long-running job blocks its own next run instead of overlapping.
- [ ] A job exceeding its timeout is killed and marked `timed_out`.
- [ ] A non-zero exit raises exactly one alert with the job name and output.
- [ ] History shows outcomes per run and is queryable per job.

## Common Pitfalls

- Assuming the previous run finished; without a lock, a slow job piles up copies of itself.
- Discarding stdout/stderr, so a failed job leaves no clue why.
- Ignoring the exit code and treating "it ran" as "it succeeded".
- Scheduling in local time and getting surprised by daylight-saving shifts — prefer UTC.

## Resources

- [crontab(5) manual](https://man7.org/linux/man-pages/man5/crontab.5.html) — the schedule expression format.
- [crontab.guru](https://crontab.guru/) — interactive cron expression reference.
- [systemd timers](https://www.freedesktop.org/software/systemd/man/systemd.timer.html) — a modern alternative with built-in logging.
- [Google SRE Book: Distributed Periodic Scheduling with Cron](https://sre.google/sre-book/distributed-periodic-scheduling/) — reliability concerns of scheduled jobs.
