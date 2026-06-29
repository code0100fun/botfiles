# Botfiles Plugin Marketplace

This repository is a Claude Code plugin marketplace. It contains curated, generalized plugins extracted from real-world projects.

## Plugins

- **zig-dev** — Zig development skills (style, memory, safety, errors, API reference)
- **elixir-phoenix** — Elixir/Phoenix development skills (framework, LiveView, Ecto, HEEx, testing, Tailwind)
- **infra-ops** — Infrastructure operations (Terraform, Docker Swarm, Ansible, deployment philosophy)
- **bees-issue-tracker** — Local-first issue tracking with the bees CLI
- **commit-message-guide** — Conventional commit messages
- **mise** — Tool-version management, task running, and env via mise.toml
- **dev-workflow** — Personal workflow composing mise + bees + commit-message-guide with Git

## Usage

Install individual plugins or the entire marketplace:

```bash
# Install from the marketplace
/plugin marketplace add code0100fun/botfiles

# Or install a single plugin
/plugin install zig-dev@botfiles
```
