# Batch Processing Script

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Processing a million records one at a time is slow and fragile; processing them all at once blows up your memory. The answer is batching: chunk the input into fixed-size groups, process each group as a unit, and commit it before moving on. Build a script that reads a large input, works through it in batches, handles the failure of a single batch without losing the ones already done, and can resume from where it stopped. This is the workhorse pattern of data engineering — the one you reach for whenever "just loop over everything" stops working.

## Prerequisites

- Comfort reading a large input file or query result iteratively
- Understanding of transactions (a batch either fully commits or rolls back)
- Basic error handling (try/except or equivalent)
- Familiarity with reading and writing a small state/checkpoint file

## Learning Objectives

By the end, you should be able to:

- Split a stream of records into fixed-size batches without loading everything
- Process each batch as an atomic unit that commits or rolls back cleanly
- Isolate a failing batch so earlier successful batches are not lost
- Checkpoint progress so an interrupted run can resume, not restart
- Report per-batch and overall progress with counts and timing
- Reason about the trade-off between batch size, memory, and throughput

## Functional Requirements

1. The script must read input and group records into batches of a configurable size.
2. Each batch must be processed transactionally: on error, that batch rolls back but the run continues.
3. A failed batch must be recorded (its identifier and reason) for later inspection or retry.
4. The script must write a checkpoint after each successful batch so it can resume from the last committed point.
5. On restart, it must skip already-processed batches and continue from the checkpoint.
6. It must report progress: batches done, records processed, failures, and total time.

## Suggested Milestones

1. **Milestone 1 — Batch & process:** Chunk the input and process each batch, printing progress.
2. **Milestone 2 — Fault isolation:** Wrap each batch in a transaction and record failures without aborting.
3. **Milestone 3 — Resume:** Add checkpointing and skip-completed logic so an interrupted run continues.

## Data & Interface Sketch

```text
input: 1,000,000 records (stream)
batch size: 500  -> 2,000 batches

per-batch flow
  accumulate 500 records
  begin transaction
    process + write batch
  commit         -> checkpoint {last_batch: N, records: N*500}
  on error       -> rollback, log {batch: N, reason}, continue

checkpoint file (checkpoint.json)
  { "last_completed_batch": 1450, "records_done": 725000 }

restart
  read checkpoint -> resume at batch 1451

CLI: process --input data.jsonl --batch-size 500 --resume
report: batches=2000 ok=1996 failed=4 records=998000 elapsed=42s
```

## Stretch Goals

- Add a `--retry-failed` mode that reprocesses only the batches recorded as failed.
- Process independent batches in parallel with a bounded worker pool.
- Make batch size adaptive, shrinking after a failure and growing when stable.
- Emit a metrics line per batch (throughput in records/sec) for tuning.

## Definition of Done

- [ ] Input is processed in configurable-size batches without loading it all.
- [ ] A single failing batch is isolated and logged; other batches still complete.
- [ ] A checkpoint is written after every successful batch.
- [ ] Killing and restarting the script resumes from the checkpoint, not the start.
- [ ] The final report shows batches, records, failures, and elapsed time.

## Common Pitfalls

- Committing per record (too slow) or per whole run (loses everything on one failure).
- Letting one bad batch throw an unhandled exception and kill the entire job.
- Checkpointing before the batch actually commits, so a crash loses committed work or double-processes.
- Choosing a batch size blindly instead of measuring memory and throughput.

## Resources

- [PostgreSQL: Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html) — the atomicity guarantee a batch relies on.
- [Python `itertools` recipes](https://docs.python.org/3/library/itertools.html#itertools-recipes) — a clean `batched()` for chunking iterables.
- [Idempotency and checkpointing](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/) — why safe resumes need idempotence.
- [Designing Data-Intensive Applications (concepts)](https://dataintensive.net/) — batch processing fundamentals in chapter 10.
