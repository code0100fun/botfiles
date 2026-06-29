---
name: mise
description: Guide for using mise (mise-en-place) — the polyglot tool-version manager, task runner, and environment manager configured via mise.toml. Use when a project has a mise.toml/.mise.toml, when installing or pinning tool versions, running project tasks (mise run / mise exec), or setting up a reproducible dev environment.
---

# mise

[mise](https://mise.jdx.dev) (mise-en-place) is a single tool that manages a project's **tool versions** (like asdf — Node, Python, Go, …), runs its **tasks** (like make/just), and sets its **environment variables** — all from one `mise.toml` at the project root. Pinning everything in `mise.toml` makes a checkout reproducible: the same versions, tasks, and env on every machine.

## mise.toml

```toml
[tools]
node = "22"
python = "3.12"
"npm:typescript" = "5"          # language packages too

[env]
DATABASE_URL = "postgres://localhost/dev"
_.path = ["node_modules/.bin"]  # prepend to PATH

[tasks.test]
run = "npm test"

[tasks.build]
run = "npm run build"
depends = ["test"]              # build runs test first
```

A new or changed config must be **trusted** before mise will use it: `mise trust`.

## Core commands

| Command | Description |
|---------|-------------|
| `mise install` | Install every tool pinned in `mise.toml` (run after cloning) |
| `mise use <tool>@<ver>` | Add/pin a tool to `mise.toml` and install it (e.g. `mise use node@22`) |
| `mise run <task>` (or `mise <task>`) | Run a task defined under `[tasks]` |
| `mise tasks` | List available tasks |
| `mise exec -- <cmd>` | Run a one-off command with mise's tools + env, no shell activation |
| `mise ls` / `mise current` | Show installed / active tool versions |
| `mise which <bin>` | Resolve which managed binary will run |
| `mise up [tool]` | Upgrade tools to the latest allowed by `mise.toml` |
| `mise trust` | Trust the current project's `mise.toml` |
| `mise activate <shell>` | Shell hook that auto-switches versions on `cd` (add to your shell rc) |

## Tasks: the project's command vocabulary

Define repeatable commands as `[tasks]` and run them with `mise run <task>`, instead of memorizing raw build/test/lint invocations. Tasks can declare `depends` (run prerequisites first) and accept arguments. `mise tasks` discovers what a project can do.

```bash
mise tasks            # what can this project do?
mise run test         # run the test task
mise run build        # depends=["test"] -> runs test, then build
```

## Conventions

1. **Every project gets a `mise.toml`** pinning its tool versions — reproducible environments over "works on my machine".
2. **Encode commands as tasks.** Build/test/lint/dev all become `[tasks]`, invoked via `mise run <task>`. Discover them with `mise tasks` before guessing raw commands.
3. **Set up after clone** with `mise install` (then `mise trust` if prompted).
4. **`mise trust` whenever the config changes** — mise refuses an untrusted `mise.toml` for safety.
5. **`mise exec -- <cmd>`** for one-offs (CI, scripts) when the shell isn't activated; otherwise `mise activate` handles version switching on `cd`.
