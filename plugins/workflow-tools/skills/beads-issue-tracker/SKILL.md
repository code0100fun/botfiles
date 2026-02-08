---
name: beads-issue-tracker
description: Guide for using Beads (bd/beads), a local-first issue tracker that stores issues as plain text files. Use when managing project issues, tracking work across sessions, or following issue-tracking workflows.
---

# Beads Issue Tracker

Beads is a minimalist, local-first issue tracker that stores issues as plain text files in a `.beads/` directory. It provides simple, file-based issue tracking without external dependencies, integrates seamlessly with git workflows, and works offline.

## Core Commands

| Command | Description |
|---------|-------------|
| `bd list` / `beads list` | List open issues |
| `bd list --all` | List all issues including closed |
| `bd list --status open` | Filter by status |
| `bd list --label TAG` | Filter by label |
| `bd show N` | Display issue #N |
| `bd ready` | Show issues with no blockers |
| `bd create --title="..." --type=TYPE --priority=N --label="x,y"` | Create new issue |
| `bd update N --status STATUS` | Update issue status |
| `bd close N` / `bd close N -r "reason"` | Close issue |
| `bd comment N "Text"` | Add comment to issue |
| `bd search "term"` | Search issues |
| `bd sync` | Sync with git |
| `bd prime` | Get compact usage examples |
| `bd onboard` | Get started guide |

## Issue Types

| Type | Usage |
|------|-------|
| `epic` | Large body of work containing multiple children |
| `feature` | New functionality being added |
| `task` | Generic work item, often implementation work |
| `bug` | Something broken that needs fixing |
| `chore` | Maintenance, cleanup, refactoring |
| `merge-request` | PR/MR tracking |

## Priority Scale

Priority uses numeric values 0-4 (or P0-P4). Do NOT use "high"/"medium"/"low":

| Priority | Meaning |
|----------|---------|
| 0 / P0 | Critical - drop everything |
| 1 / P1 | High - do next |
| 2 / P2 | Medium - normal work |
| 3 / P3 | Low - when time permits |
| 4 / P4 | Backlog - someday/maybe |

## What is "Non-Trivial"?

- **Trivial** (no issue needed): typo fixes, single-line changes, adding one test
- **Non-trivial** (issue required): refactoring, new features, new CLI flags, bug fixes, multi-file changes
- **Rule of thumb**: If it takes >5-10 minutes or spans multiple files, create an issue

## Workflow

### Starting a Session

```bash
bd ready                    # Show issues with no blockers
bd list --status=open       # All open issues
bd show <id>                # View issue details, description, and history
```

### Working on an Issue

```bash
bd update <id> --status in_progress   # Claim work
# ... do the work ...
bd close <id> -r "Implemented feature X"  # Complete work
```

### If You Need to Pause

```bash
bd update <id> --status open   # Release the issue
```

### Creating Issues

```bash
bd create --title="Add params support" --type=feature --priority=2 --label="api,core"
```

## Session Close Protocol

At the end of each session:

1. **Review active issues**: `bd list --status open`
2. **Close completed issues**: `bd close N -r "reason"`
3. **Add progress comments**: `bd comment N "In progress: completed X, next is Y"`
4. **Create new issues for discovered work**: `bd create --title="Follow-up task" ...`
5. **Document decisions**: `bd comment N "Decided approach A because of constraint B"`
6. **Verify state**: `bd list` should show accurate project state
7. **Commit beads changes**: `git add .beads/ && git commit`

## Important Rules

- **NEVER edit `.beads/issues.jsonl` manually** - always use `bd` commands
- Only have ONE issue `in_progress` at a time per epic
- Close child issues before closing parent epics
- **Do NOT auto-push** - only commit/push when explicitly asked
- Label tasks with relevant labels for grouping; try to use existing labels when possible

## Best Practices

### Issue Creation
- **Be specific**: "Add PostgreSQL connection pooling" not "Fix database"
- **Single responsibility**: One issue per logical task
- **Add labels**: Use labels to categorize (bug, feature, docs, refactor)
- **Include context**: Add relevant details in the description

### Issue Management
- **Update frequently**: Add comments as work progresses
- **Close promptly**: Close issues when work is complete
- **Reference commits**: Link to commits in comments
- **Track blockers**: Document what's blocking progress

### Common Labels
- **Type**: `bug`, `feature`, `enhancement`, `docs`, `refactor`, `test`
- **Component**: `api`, `database`, `frontend`, `infra`, `cli`, `core`
