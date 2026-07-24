# CLI User Manager

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Beginner · **Estimated time:** 4–7 hours

## Overview

Build a command-line tool that manages a list of users — add, list, update, delete, and search — persisting them to a local file. There is no web server here; the interface is the terminal. This project teaches the other half of backend work: parsing arguments, giving clear feedback through exit codes and output, and treating a plain file as a small, reliable data store. It is the kind of internal tool backend engineers write constantly.

## Prerequisites

- Basic programming in your chosen language and how to run a script from the terminal
- Understanding of JSON or CSV as a storage format
- Familiarity with reading and writing files
- A CLI argument-parsing library (argparse/Click/Typer, Commander, cobra) or the will to parse `argv` yourself

## Learning Objectives

By the end, you should be able to:

- Design a subcommand-based CLI (`tool add`, `tool list`, ...) with flags and arguments
- Validate input and report errors with meaningful, non-zero exit codes
- Persist structured records to a file and read them back reliably
- Enforce a uniqueness constraint (e.g. unique email) across records
- Format terminal output so it is readable by both humans and scripts

## Functional Requirements

1. The tool must support adding a user with at least a name and an email.
2. The tool must reject a user whose email is malformed or already exists, with a clear message and non-zero exit.
3. The tool must list all users in a readable format.
4. The tool must update and delete a user identified by a stable key (ID or email).
5. The tool must search or filter users by a field (name or email substring).
6. All changes must persist to a file and be visible on the next invocation.
7. The tool must exit 0 on success and non-zero on any handled error.

## Suggested Milestones

1. **Milestone 1 — Add & list:** Parse an `add` command, append to the file, and implement `list`.
2. **Milestone 2 — Update, delete, search:** Round out the CRUD verbs plus a search/filter command.
3. **Milestone 3 — Validation & UX:** Enforce email format and uniqueness, add helpful `--help`, and use correct exit codes.

## Data & Interface Sketch

```text
Storage: users.json  ->  [ { id, name, email, role, createdAt } ]

tool add    --name "Ada" --email ada@example.com [--role user]
tool list   [--role admin] [--sort name]
tool update --id u_01 --name "Ada L."
tool delete --id u_01
tool search --email ada

Exit codes: 0 ok | 1 validation error | 2 not found | 3 storage error
```

## Stretch Goals

- Add an interactive mode with prompts when required flags are omitted.
- Add colored output and a table layout for `list`.
- Support export to CSV and import back from it.
- Add a `--json` flag so output is machine-readable for piping.

## Definition of Done

- [ ] Every subcommand works and persists changes across separate invocations.
- [ ] Adding a duplicate or invalid email fails clearly with a non-zero exit code.
- [ ] `--help` describes every command and flag.
- [ ] Deleting or updating a missing user returns a not-found error, not a crash.
- [ ] Success and failure map to correct, documented exit codes.

## Common Pitfalls

- Printing errors to stdout instead of stderr, which breaks piping and scripting.
- Always exiting 0, so callers cannot tell success from failure.
- Rewriting the whole file non-atomically and losing data if the process is interrupted.
- Treating the array index as the user ID, which shifts after deletes.

## Resources

- [Command Line Interface Guidelines](https://clig.dev/) — modern, practical CLI design principles.
- [Python argparse tutorial](https://docs.python.org/3/howto/argparse.html) — the standard library approach for Python.
- [Wikipedia: Exit status](https://en.wikipedia.org/wiki/Exit_status) — the convention behind exit codes.
- [The Art of Command Line](https://github.com/jlevy/the-art-of-command-line) — broader terminal fluency.
