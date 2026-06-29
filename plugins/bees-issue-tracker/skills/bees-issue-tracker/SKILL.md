---
name: bees-issue-tracker
description: Guide for using bees, a local-first issue tracker (SQLite DB + a committed issues.jsonl) driven by the `bees` CLI. Use for ALL task/issue tracking on bees projects — creating issues, organizing them into epics with dependencies, the in_progress→closed lifecycle, and syncing/committing the tracker.
---

# Bees Issue Tracker

`bees` is a local-first issue tracker. Issues live in a SQLite DB at `.bees/bees.db` (gitignored) and are exported to `.bees/issues.jsonl`, which **is committed** — that jsonl is the tracker's source of truth in git.

On a project that uses bees, **bees is the ONLY task tracker** — never use the harness `TaskCreate`/`TodoWrite` (or any todo) tools to track work.

> The binary is `bees` — invoke it directly (it must be installed and on your PATH).

> Watch the bees CLI specifics: titles are **positional**, priorities are **1–4** (there is no P0), types are `task|bug|feature|epic|story`, and labels are managed via a subcommand (not a `create` flag). The tables below are authoritative for bees 0.4.

## Start each session: `bees prime`

Run **`bees prime`** at the start of a session — it dumps the workflow context + issues for the agent, re-establishing tracker state and conventions. Then orient:

```bash
bees prime                 # workflow context + issues (run first)
bees ready                 # open issues with no unresolved blockers
bees list -s open          # all open issues
bees show <id>             # one issue: description, children, deps, comments
```

## Core commands

| Command | Description |
|---------|-------------|
| `bees prime` | Dump workflow context for AI agents (run at session start) |
| `bees ready [--json]` | Open issues with no unresolved blocking deps |
| `bees list [-s <status>] [-p <N>] [-a <name>] [--json]` | List issues (alias `ls`) |
| `bees show <id>` | Issue details: description, children, deps, comments |
| `bees create "<title>" -t <type> -p <N> [-d <desc>] [--acceptance <text>] [--design <text>]` | Create an issue (title is **positional**) |
| `bees update <id> -s <status>` | Update status (also `--title`, `-p`, `-a`, `-d`, `-t`, `--acceptance`, `--notes`, `--design`, `--due`, `--defer`) |
| `bees close <id> [--reason "<text>"]` | Close an issue |
| `bees comment add <id> "<text>"` · `bees comment list <id>` | Comments |
| `bees dep add <child> <parent> -t parent-child` | Add a dependency (also `blocks`, `related`); plus `dep remove`, `dep list <id>` |
| `bees label add <id> <label>` · `bees label remove <id> <label>` | Manage labels (NOT a `create` flag) |
| `bees edit <id>` | Edit fields in `$EDITOR` |
| `bees sync` | Export DB → `issues.jsonl` (run before committing) |
| `bees import` | Rebuild DB from `issues.jsonl` |
| `bees rename-prefix <old> <new>` | Rename the id prefix across all issues |
| `bees init` · `bees config` · `bees daemon start\|stop\|status` | Setup / config / background daemon |

There is **no `search` or `onboard`** subcommand — filter with `bees list`, read with `bees show`, or grep `.bees/issues.jsonl`.

## Issue types

`task` · `bug` · `feature` · `epic` · `story`. (No `chore`/`merge-request` — use `task`.)

## Priority: 1–4 (1 = critical)

bees priorities are **1–4** — there is no P0/0. Never pass "high"/"medium"/"low" as the value.

| Priority | Meaning |
|----------|---------|
| 1 / P1 | Critical — drop everything |
| 2 / P2 | High — do next |
| 3 / P3 | Medium — normal work |
| 4 / P4 | Low / backlog |

## Statuses

`open` → `in_progress` → `closed`, plus `deferred` (snooze with `--defer <date>`).

## How I want bees used (conventions)

These conventions are the point of this skill — follow them on bees projects:

1. **bees only.** All task tracking goes through bees, never the harness todo/Task tools. Begin sessions with `bees prime`.
2. **Per-project prefix.** Issue ids carry a project prefix, e.g. `proj-N` (`starside-N` on the Starside project). Change it with `bees rename-prefix`.
3. **Issue before code.** Create the issue *before* starting non-trivial work: `bees create "<title>" -t task -p N`.
4. **Set `in_progress` when you START — not just open→closed.** Run `bees update <id> -s in_progress` *before* writing code, so the tracker always reflects what's actively being worked. Keep one issue in_progress at a time.
5. **Phases are epics; nest with deps.** Model a phase/milestone as an `epic` and nest its tasks under it: `bees dep add <child-task> <parent-epic> -t parent-child` (**child first, parent second**). Do **not** also add `phase-N` labels — the epic + nesting already convey the phase. Pick one mechanism, not both.
6. **Labels = cross-cutting area only.** Use labels for area/component (`auth`, `cli`, `manifest`, `godroll`, …), not for phase or status. Add via `bees label add <id> <label>`.
7. **Close → sync → commit the jsonl.** On completion: `bees close <id> --reason "..."`, then `bees sync`, then **commit the `.bees/issues.jsonl` change** (e.g. `chore(bees): close proj-N …`). Never leave closed-issue jsonl edits uncommitted.
8. **Per-task cadence.** Each task flows: `in_progress` → build/verify → commit the **code** → `close` → commit the **bees jsonl**. Keep tracker commits separate from code commits.
9. **Close the epic** once all its children are closed (phase complete), and commit that jsonl change too.

## Git & state

- **Committed:** `.bees/issues.jsonl` (the tracker's source of truth in git).
- **Gitignored (generated):** `.bees/bees.db`, `.bees/bees.db-*`.
- `bees update`/`close` write the DB only — run **`bees sync`** to export to `issues.jsonl` before you `git add` it.
- **Never hand-edit `issues.jsonl`** — use `bees` commands, then `bees sync`.
- Don't auto-push. Commit only when the user asks (the per-task cadence above applies once the user has opted into committing per task).

## What needs an issue?

- **Trivial (no issue):** typo, one-line change, adding a single test.
- **Non-trivial (issue first):** features, refactors, new CLI flags/commands, bug fixes, multi-file changes. Rule of thumb: **>5–10 min of work or spanning multiple files → create an issue.**

## Editor integration

Prefer a GUI alongside the CLI? The **bees VS Code extension** (https://github.com/ctxshift/vscode-bees) surfaces the tracker in-editor. The `bees` CLI and the conventions above remain the source of truth.
