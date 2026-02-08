---
name: session-completion
description: Protocol for cleanly ending a work session including filing issues, running quality gates, updating status, committing, and handing off context. Use when wrapping up a coding session.
---

# Session Completion (Landing the Plane)

**When ending a work session**, complete these steps to ensure clean handoff and no lost context.

## Workflow

### 1. File Issues for Remaining Work

Create issues for anything that needs follow-up:
- Bugs discovered but not fixed
- Features partially implemented
- Technical debt identified
- Ideas that came up during the session

### 2. Run Quality Gates (if code changed)

Run the project's quality checks:
- Tests (`mix test`, `zig build test`, `npm test`, etc.)
- Linters and formatters
- Build verification
- Any project-specific precommit checks

### 3. Update Issue Status

- Close finished work with a reason/summary
- Update in-progress items with progress notes
- Document decisions and blockers in comments

### 4. Commit Changes

```bash
git add <specific files>
git commit -m "message"
```

If using an issue tracker that stores files in the repo (e.g., Beads), sync those changes too.

### 5. Hand Off Context

Provide context for the next session:
- What was completed
- What's still in progress
- Any blockers or decisions needed
- Recommended next steps

## Critical Rules

- **DO NOT** run `git push` automatically - the user will push manually
- **DO NOT** run `git commit` automatically - only commit when the user explicitly asks
- If SSH authentication is required, do not retry - just note it for the user
- Commit frequently with meaningful messages
- Keep issue tracker changes separate from code changes when possible

## Session Summary Template

```
Session ending. Here's the status:

1. Completed:
   - [What was finished]

2. In Progress:
   - [What's partially done, with context]

3. Issues Filed:
   - [New issues created for follow-up]

4. Next Steps:
   - [What to focus on next session]
```

## Why This Matters

Clean session completion ensures:
- No work is lost or forgotten
- The next session (or team member) has full context
- Issue trackers reflect actual project state
- Code is committed and safe
- Quality gates catch issues before they compound
