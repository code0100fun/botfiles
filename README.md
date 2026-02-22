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

### workflow-tools
Developer workflow skills covering commit message conventions, Beads issue tracking, and session completion protocols.

**Skills:** `commit-message-guide` `beads-issue-tracker` `session-completion`

## Install

```bash
# Add the marketplace
/plugin marketplace add code0100fun/botfiles

# Then install individual plugins
/plugin install code0100fun/botfiles/plugins/zig-dev
/plugin install code0100fun/botfiles/plugins/elixir-phoenix
/plugin install code0100fun/botfiles/plugins/infra-ops
/plugin install code0100fun/botfiles/plugins/workflow-tools
```

## Structure

```
botfiles/
├── .claude-plugin/marketplace.json
├── CLAUDE.md
└── plugins/
    ├── zig-dev/           (5 skills)
    ├── elixir-phoenix/    (7 skills)
    ├── infra-ops/         (4 skills)
    └── workflow-tools/    (3 skills)
```

Each plugin contains a `.claude-plugin/plugin.json` manifest and a `skills/` directory with individual `SKILL.md` files.
