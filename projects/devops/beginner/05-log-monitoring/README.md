# Log Monitoring Script

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build a script that watches an application's log files, spots trouble as it happens, and tells someone before a user has to. It tails the log, matches lines against patterns you define (errors, stack traces, slow requests), counts them over a window, and raises an alert when a threshold is crossed. The craft is in the details: following a file that rotates out from under you, avoiding a flood of duplicate alerts, and tuning thresholds so real problems fire but ordinary noise stays quiet. When you finish, you will have the small, dependable eyes-on-the-logs that every service needs before it grows a full observability stack.

## Prerequisites

- An app that writes logs to a file (or a sample log you can append to)
- A scripting language comfortable with text (Python, Bash + tools, or similar)
- Understanding of regular expressions for pattern matching
- A place to send alerts (console, file, or a chat webhook)

## Learning Objectives

By the end, you should be able to:

- Follow an actively-written log file, including across rotation
- Match lines against configurable patterns and classify severity
- Count matches within a time window and alert on a threshold
- Suppress duplicate alerts so one incident does not spam
- Tune thresholds to balance sensitivity against false positives

## Functional Requirements

1. The script must follow a log file in real time and process new lines as they arrive.
2. It must match lines against a configurable list of patterns with severities.
3. It must count matches over a rolling time window per pattern.
4. It must raise an alert when a pattern's count exceeds its threshold in the window.
5. It must deduplicate or rate-limit alerts so one condition alerts once, not per line.
6. It must keep following the file correctly after log rotation.
7. Patterns, thresholds, and alert destinations must be configurable, not hardcoded.

## Suggested Milestones

1. **Milestone 1 — Tail & match:** Follow the file and print lines that match any configured pattern.
2. **Milestone 2 — Window & threshold:** Count matches over a window and alert when a threshold is crossed.
3. **Milestone 3 — Dedup & rotation:** Rate-limit repeated alerts and handle a rotated log without missing lines.

## Data & Interface Sketch

```text
Config (structure, not full file)
  log_file: /var/log/app/app.log
  window_seconds: 60
  patterns:
    - name: error_spike
      regex: "ERROR|FATAL"
      severity: high
      threshold: 10          # matches per window
    - name: slow_request
      regex: "duration_ms=([0-9]{4,})"
      severity: medium
      threshold: 5
  alert:
    channel: webhook|console|file
    cooldown_seconds: 300    # dedup window per pattern

Alert payload
  { pattern, severity, count, window, sample_line, timestamp }
```

## Stretch Goals

- Extract numeric fields (e.g. latency) and alert on an average or percentile, not just a count.
- Summarize the top error signatures over the last hour on demand.
- Support multiple log files and tag alerts with their source.
- Add a `--since` replay mode to test rules against historical logs.

## Definition of Done

- [ ] New log lines are processed within a second of being written.
- [ ] Crossing a configured threshold produces exactly one alert per cooldown window.
- [ ] Rotating the log (rename + recreate) does not stop monitoring or duplicate lines.
- [ ] Patterns and thresholds can be changed via config without editing code.
- [ ] Ordinary log noise below threshold produces no alerts.

## Common Pitfalls

- Re-reading the whole file on each poll instead of tracking position, wasting CPU and re-alerting.
- Holding the old file handle after rotation and silently monitoring a file no one writes to anymore.
- Alerting per matching line, burying the real signal under hundreds of duplicates.
- Writing greedy regexes that match far more than intended and inflate counts.

## Resources

- [The Art of Monitoring (concepts)](https://artofmonitoring.com/) — thresholds, signals, and alert design.
- [logrotate manual](https://man7.org/linux/man-pages/man8/logrotate.8.html) — how rotation works, so you can survive it.
- [MDN: Regular expressions guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions) — pattern matching fundamentals.
- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) — what is worth alerting on.
