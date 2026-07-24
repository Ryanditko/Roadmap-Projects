# Backup Automation

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build a script that backs up what matters — a database dump, a directory of files, some config — on a schedule, and just as importantly, proves the backup is restorable. A backup you have never restored is a hope, not a safety net. You will compress and timestamp each backup, apply a retention policy so old copies are pruned instead of filling the disk, and run a restore into a throwaway location to verify the archive is intact. Along the way you will meet the 3-2-1 rule and the uncomfortable truth that the hard part of backups is always the restore.

## Prerequisites

- Something worth backing up (a database, a data directory, or config files)
- A scripting language or shell with access to the source and a target location
- Understanding of compression and file archives (tar, zip)
- A destination with enough space (local disk, external drive, or object storage)

## Learning Objectives

By the end, you should be able to:

- Automate a scheduled, timestamped backup of a defined source
- Compress archives and name them so they sort and prune cleanly
- Apply a retention policy that keeps recent backups and removes old ones
- Verify a backup by restoring it to a temporary location
- Alert when a backup or verification fails

## Functional Requirements

1. The script must back up a configured source (files and/or a database dump) on demand and on a schedule.
2. Each backup must be compressed and named with a sortable UTC timestamp.
3. A retention policy must keep the last N (or N days of) backups and delete older ones.
4. The script must verify each backup, at minimum by testing the archive's integrity.
5. A restore path must exist and be exercised into a temporary location, not over live data.
6. The script must exit non-zero and alert on any backup or verification failure.
7. Source, destination, retention, and schedule must be configurable.

## Suggested Milestones

1. **Milestone 1 — Snapshot:** Archive the source into a compressed, timestamped file on demand.
2. **Milestone 2 — Retention & schedule:** Prune old backups by policy and run the backup on a schedule.
3. **Milestone 3 — Verify & restore:** Test archive integrity and perform a restore into a scratch location.

## Data & Interface Sketch

```text
Config (structure, not full file)
  source:
    files: [/etc/app, /srv/app/data]
    database: { type: postgres, name: appdb }   # dumped, not raw files
  destination: /backups            (or s3://bucket/prefix)
  retention: { keep_days: 7, keep_min: 3 }
  schedule: "0 2 * * *"            # cron expression, external scheduler

Artifact naming
  app-backup-YYYYMMDDTHHMMSSZ.tar.gz

Flow (structure, not full script)
  dump db -> archive files + dump -> compress -> write to destination
  verify: test archive integrity (and optional restore to /tmp/verify)
  prune: list backups -> keep by retention -> delete the rest
  on any failure: log + alert + exit non-zero
```

## Stretch Goals

- Encrypt archives at rest and manage the key outside the backup itself.
- Add incremental or differential backups to shrink daily size.
- Push a copy off-site (object storage) to satisfy the 3-2-1 rule.
- Emit a metric or report: last successful backup time and total size.

## Definition of Done

- [ ] A backup runs unattended on schedule and produces a timestamped, compressed archive.
- [ ] Retention prunes old backups without ever deleting the most recent valid one.
- [ ] Archive integrity is verified after each run.
- [ ] A restore into a temporary location reproduces the original data.
- [ ] A failed backup or verification exits non-zero and raises an alert.

## Common Pitfalls

- Backing up raw database files while the DB is running, producing an inconsistent snapshot — dump instead.
- Never testing a restore, so the first real restore is also the first time you learn it does not work.
- A retention bug that deletes everything, including the backup you need, when the list is empty or misparsed.
- Storing backups on the same disk as the source, so one disk failure takes both.

## Resources

- [US-CERT: Data Backup Options (3-2-1 rule)](https://www.cisa.gov/sites/default/files/publications/data_backup_options.pdf) — the foundational strategy.
- [GNU tar manual](https://www.gnu.org/software/tar/manual/tar.html) — archiving and incremental backups.
- [PostgreSQL: Backup and Restore](https://www.postgresql.org/docs/current/backup.html) — consistent database dumps.
- [restic documentation](https://restic.readthedocs.io/) — a modern tool that models retention and verification well.
