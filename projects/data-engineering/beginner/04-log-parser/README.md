# Log Parser

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Server logs arrive as walls of text, but hidden inside each line is a structured record: a timestamp, a level, a source, a message. Build a tool that reads a log file, extracts those fields into clean records, and lets you filter and summarize them — how many errors in the last hour, which endpoint is slowest, what the top messages are. This is the foundational skill behind every observability pipeline: turning free-form text into queryable data. The heart of the project is a robust parser that handles the lines that *almost* match the format without falling over.

## Prerequisites

- Comfort reading files line by line
- Beginner-level regular expressions or knowledge of a structured log format (JSON lines)
- Basic understanding of timestamps and time ranges
- Familiarity with dictionaries/maps for grouping and counting

## Learning Objectives

By the end, you should be able to:

- Define a log line format and parse it into named fields
- Handle both a fixed text format (via regex) and structured JSON lines
- Deal gracefully with lines that do not match — count them, don't crash
- Filter records by level, time window, or field value
- Aggregate: counts by level, top-N messages, requests per minute
- Stream a large file without loading it entirely into memory

## Functional Requirements

1. The tool must parse each line into a record with at least timestamp, level, and message.
2. It must support a configurable line format (regex pattern or JSON) rather than hard-coding one.
3. Lines that fail to parse must be counted and optionally written to an "unparsed" file, never dropped silently.
4. The tool must filter records by level and by a time range given on the command line.
5. It must produce aggregations: count per level and the top-N most frequent messages.
6. It must process files line by line so arbitrarily large logs are handled.

## Suggested Milestones

1. **Milestone 1 — Parse:** Turn each matching line into a structured record and print it.
2. **Milestone 2 — Filter:** Add level and time-range filtering over the parsed records.
3. **Milestone 3 — Aggregate & report:** Add counts per level, top-N messages, and unparsed-line handling.

## Data & Interface Sketch

```text
source line (Apache-ish)
  2024-05-01T10:15:03Z ERROR api order-service "db timeout" 503

parse pattern (regex, named groups)
  (?P<ts>\S+) (?P<level>\w+) (?P<comp>\S+) (?P<svc>\S+) "(?P<msg>[^"]*)" (?P<code>\d+)

structured record
  { ts, level, component, service, message, status }

unparsed lines -> unparsed.log (with line number)

query flow
  read -> parse -> [ok] filter(level>=WARN, ts in window) -> aggregate
                \-> [no match] count + unparsed sink

report
  by_level: INFO=8210 WARN=145 ERROR=37
  top_messages: "db timeout" x22, "retry" x15
```

## Stretch Goals

- Auto-detect the format (JSON vs text) by sniffing the first lines.
- Add a `--follow` mode that tails the file and parses new lines as they arrive.
- Compute latency percentiles if a duration field is present.
- Support gzip-compressed log files transparently.

## Definition of Done

- [ ] Well-formed lines parse into records with all expected fields.
- [ ] Unmatched lines are counted and preserved, not silently discarded.
- [ ] Level and time-range filters return the correct subset.
- [ ] Counts per level and top-N messages are accurate.
- [ ] A multi-hundred-MB file processes without exhausting memory.

## Common Pitfalls

- A brittle regex that breaks on quoted fields, extra spaces, or multiline messages.
- Parsing timestamps without handling timezones, so time-range filters are off.
- Loading the whole file into a list before processing large logs.
- Silently skipping unparsed lines and hiding a format drift you should know about.

## Resources

- [Python `re` module](https://docs.python.org/3/library/re.html) — named groups make log patterns readable.
- [regex101](https://regex101.com/) — build and test log patterns interactively.
- [The Twelve-Factor App: Logs](https://12factor.net/logs) — why treating logs as event streams matters.
- [Elastic: Grok processor](https://www.elastic.co/guide/en/elasticsearch/reference/current/grok-processor.html) — how a production system names log-field patterns.
