# Botfiles

A curated collection of [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugins extracted and generalized from real-world projects.

## Plugins

### zig-dev
Zig development skills covering style enforcement, memory leak detection, safety checking, error handling validation, and Zig 0.15 ArrayList API reference.

**Skills:** `zig-style-enforcer` `memory-leak-detector` `safety-checker` `error-handling-validator` `zig-arraylist`

### elixir-phoenix
Elixir and Phoenix development skills covering framework conventions (Phoenix 1.8), LiveView patterns (streams, JS interop, forms), Ecto best practices, HEEx template rules, testing with Ecto.SQL.Sandbox, and Tailwind CSS v4.

**Skills:** `phoenix-guidelines` `elixir-guidelines` `ecto-guidelines` `liveview-guidelines` `heex-templates` `elixir-testing` `tailwind-v4`

### infra-ops
Infrastructure operations skills covering Terraform safety practices, Docker Swarm deployment patterns, Ansible operations with secret management, and repeatable deployment philosophy.

**Skills:** `terraform-safety` `docker-swarm` `ansible-ops` `repeatable-deployments`

### bees-issue-tracker
Local-first issue tracking with the `bees` CLI — issues as a committed `issues.jsonl`, epics with dependencies, and the `in_progress` → `closed` lifecycle.

**Skills:** `bees-issue-tracker`

### commit-message-guide
Well-structured conventional commits — `type(scope): subject` in imperative mood, bodies only when they add value, and no AI attribution.

**Skills:** `commit-message-guide`

### mise
Using [mise](https://mise.jdx.dev) (mise-en-place) for tool-version management, task running, and environment config via `mise.toml`.

**Skills:** `mise`

### dev-workflow
A personal cross-codebase development workflow that composes the `mise`, `bees-issue-tracker`, and `commit-message-guide` skills with Git. Installing it pulls those three in automatically (declared as plugin dependencies).

**Skills:** `dev-workflow`

## Install

Add the marketplace once:

```bash
/plugin marketplace add code0100fun/botfiles
```

### The full dev workflow (with dependencies)

`dev-workflow` declares `mise`, `bees-issue-tracker`, and `commit-message-guide` as **plugin dependencies**, so installing it pulls all three in automatically (and reports what it added) — one command gets you the whole workflow:

```bash
/plugin install dev-workflow@botfiles
#  ↳ also installs: mise, bees-issue-tracker, commit-message-guide
```

### Individual skills (atomic)

Every plugin installs standalone, so you can take just what you want with none of the workflow attached — e.g. the issue tracker on its own:

```bash
/plugin install bees-issue-tracker@botfiles
```

Any of the plugins can be installed the same way:

```bash
/plugin install commit-message-guide@botfiles
/plugin install mise@botfiles
/plugin install zig-dev@botfiles
/plugin install elixir-phoenix@botfiles
/plugin install infra-ops@botfiles
```

Dependencies only flow one way: installing `dev-workflow` pulls in its companions, but installing an atomic plugin like `bees-issue-tracker` never drags in the workflow. To remove the workflow **and** the companions it pulled in: `claude plugin uninstall dev-workflow --prune`.

## Structure

```
botfiles/
├── .claude-plugin/marketplace.json
├── CLAUDE.md
└── plugins/
    ├── zig-dev/                (5 skills)
    ├── elixir-phoenix/         (7 skills)
    ├── infra-ops/              (4 skills)
    ├── bees-issue-tracker/     (1 skill)
    ├── commit-message-guide/   (1 skill)
    ├── mise/                   (1 skill)
    └── dev-workflow/           (1 skill)
```

Each plugin contains a `.claude-plugin/plugin.json` manifest and a `skills/` directory with individual `SKILL.md` files.
