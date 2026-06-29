---
name: dev-workflow
description: My personal cross-codebase development workflow — set up tooling with mise, track work in Bees, make focused changes, commit with the commit-message-guide conventions, and land the plane cleanly. Use when starting or running a development task on a project that follows this workflow (mise + Bees + Git).
---

# Dev Workflow

My default way of working on almost any codebase. It **composes three companion skills** — `mise`, `bees-issue-tracker`, and `commit-message-guide` — which install automatically as dependencies of this plugin. This skill orchestrates them; it does not restate them.

## 1. Set up tooling — mise

Use the **mise** skill. After cloning, run `mise install` (and `mise trust` if prompted) to get the pinned tool versions. Discover the project's commands with `mise tasks` and run everything through `mise run <task>` rather than raw invocations.

## 2. Track the work — Bees

Use the **bees-issue-tracker** skill. Start with `bees prime`. Create an issue **before** non-trivial work, and set it `in_progress` when you actually start — keep one issue in_progress at a time.

## 3. Make the change

Work in small, verifiable increments. Run the project's checks via mise tasks (`mise run test`, `mise run lint`) as you go — don't let a change pile up unverified.

## 4. Commit — commit-message-guide

Use the **commit-message-guide** skill to draft the message (`type(scope): subject`, imperative mood, a body only when it adds value, no AI attribution). Stage specific files. Keep **code commits separate from tracker commits**. Do **not** `git commit` or `git push` automatically — only when I ask.

## 5. Land the plane

Wrap up so the next session has full context:

- **Update Bees:** `bees close <id> --reason "…"` for finished work (or add progress notes to in_progress items), then `bees sync` and commit the `.bees/issues.jsonl` change — separately from code.
- **Summarize:** what's **done**, what's **in progress** (with context), and the recommended **next steps**.
- **Flag** any blockers or decisions needed.

## Critical rules

- **Never auto-commit or auto-push** — commit only when I ask; never push.
- **Code and tracker changes are separate commits.**
- **One issue `in_progress` at a time** — the tracker should always reflect what's actively being worked.
- Prefer `mise run <task>` over remembering raw commands.
