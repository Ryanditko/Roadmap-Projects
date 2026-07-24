# Basic Scheduler (cron-like)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Data pipelines rarely run on demand — they run *on a schedule*. Build a small cron-like scheduler that reads a list of jobs, each with a time expression, and runs each one when its time comes. You will parse cron-style expressions, compute the next run time, execute the due jobs, and keep a history of what ran and whether it succeeded. The deceptively hard parts are the ones every real scheduler wrestles with: not running a job twice for the same tick, deciding what to do about a missed run, and making sure a slow job does not silently overlap itself.

## Prerequisites

- Comfort working with dates, times, and the concept of "now"
- Basic parsing of a structured string (a cron expression)
- Ability to execute a function or shell command from code
- Beginner file I/O for reading job definitions and writing history

## Learning Objectives

By the end, you should be able to:

- Parse a cron-style expression into a schedule (minute, hour, day, month, weekday)
- Compute the next scheduled run time from a given moment
- Run a main loop that fires due jobs without busy-waiting
- Guarantee a job runs once per scheduled tick, not zero or twice
- Record execution history with status, start, and duration
- Prevent a long-running job from overlapping its next invocation

## Functional Requirements

1. The scheduler must load a list of jobs, each with a cron-style expression and a command/function to run.
2. It must correctly parse the expression and compute each job's next run time.
3. The main loop must execute a job exactly once when its scheduled time arrives.
4. It must never double-fire a job for the same tick, even if the loop is slightly delayed.
5. If a run is still in progress when the next is due, the scheduler must skip or queue it per a defined policy (no silent overlap).
6. It must persist a run history: job name, scheduled time, start, end, and success/failure.

## Suggested Milestones

1. **Milestone 1 — Parse & compute:** Parse a cron expression and print the next N run times.
2. **Milestone 2 — Run loop:** Add a loop that fires jobs at their due time and logs each run.
3. **Milestone 3 — Correctness:** Add once-per-tick guarantees, overlap prevention, and persisted history.

## Data & Interface Sketch

```text
job definitions (jobs file)
  daily_export   `0 2 * * *`   -> run export.job
  every_15m      "*/15 * * * *"-> run sync.job
  weekday_report "30 8 * * 1-5"-> run report.job

cron field order
  minute hour day-of-month month day-of-week

scheduler loop
  now = current time (truncated to the minute)
  for job in jobs:
    if job.next_run <= now and job.last_tick != now:
      run(job); record(status, start, duration)
      job.last_tick = now
      job.next_run  = compute_next(job.expr, now)
    if job still running and due again -> skip (policy)
  sleep until next minute boundary

history.log
  daily_export 2024-05-01T02:00 ok  duration=41s
  sync         2024-05-01T02:15 fail duration=3s "timeout"
```

## Stretch Goals

- Add catch-up policy: on startup, decide whether to run jobs that were missed while down.
- Support time zones so `0 2 * * *` means 2 AM in a configured zone.
- Add per-job retry with a max attempt count on failure.
- Expose a small status command listing each job's last and next run.

## Definition of Done

- [ ] Cron expressions parse and next-run computation matches hand-checked cases.
- [ ] A due job runs exactly once at its scheduled tick.
- [ ] The loop never double-fires a job for the same minute.
- [ ] An overlapping run is skipped or queued per the documented policy.
- [ ] Run history records status, timing, and failures for every execution.

## Common Pitfalls

- Busy-waiting in a tight loop instead of sleeping until the next boundary, burning CPU.
- Double-firing because the loop checks the same minute twice — track the last tick per job.
- Ignoring timezones and DST, so schedules drift or fire twice on clock changes.
- Letting a slow job overlap its next run, so two copies mutate the same data at once.

## Resources

- [crontab.guru](https://crontab.guru/) — interactively decode and verify cron expressions.
- [Wikipedia: Cron](https://en.wikipedia.org/wiki/Cron) — the field format and its history.
- [Python `datetime`](https://docs.python.org/3/library/datetime.html) — computing next-run times and durations correctly.
- [croniter library](https://github.com/kiorky/croniter) — a reference implementation of next-run computation to compare against.
