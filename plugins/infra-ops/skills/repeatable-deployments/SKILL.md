---
name: repeatable-deployments
description: Philosophy and practices for ensuring all infrastructure changes are codified, reproducible, and validated. Use when making infrastructure changes, deploying services, or reviewing operational procedures.
---

# Repeatable Deployments

**All infrastructure changes MUST be codified in the repository.** Manual changes to servers are temporary fixes that will be lost on redeployment. If you make a manual change to fix something, you MUST then codify it.

## What Must Be in the Repo

1. **Configuration files** - Any config files placed on servers (model configs, app configs, etc.)
   - Store in appropriate directories with clear naming
   - Deploy via automation tasks that copy files to target locations

2. **Environment variables** - All env vars in compose templates, secrets via secret management lookups
   - Never hardcode secrets; use secret management integration (1Password, Vault, etc.)

3. **Directory structure** - Any directories that must exist on target hosts
   - Create via automation tasks with correct ownership/permissions

4. **External dependencies** - Backends, plugins, or binaries extracted from images
   - Document extraction steps in automation comments
   - Add tasks to extract/install them

5. **Permissions** - File/directory permissions that differ from defaults
   - Explicitly set `mode`, `owner`, `group` in automation tasks

## Anti-Patterns (DO NOT DO)

- SSH into a server and edit files directly without codifying the change
- Manually `docker exec` to modify container state
- Creating directories or configs via ad-hoc commands without adding to automation
- Fixing permissions manually without codifying the fix
- Saying "I fixed it" without committing the fix to the repo

## Validation Workflow (REQUIRED)

**Work is NOT complete until it can be reproduced from scratch.** Manual fixes during debugging are acceptable, but the final deliverable must be validated by:

1. **Codify all changes** - Update automation scripts/IaC with the fix
2. **Tear down the feature** - Remove the manually-configured state
   - Delete extracted binaries/backends
   - Remove deployed configs
   - Stop/remove the service
3. **Redeploy from scratch** - Run the automation with no manual intervention
4. **Verify functionality** - Test that the feature works as expected
5. **Only then commit** - The automation is proven to work

## Why This Matters

This ensures:
- New team members can deploy without tribal knowledge
- Disaster recovery is possible (node dies, rebuild from scratch)
- Changes are reviewable in git history
- No "works on my machine" situations

**Exception**: Quick debugging sessions may skip validation, but must create a follow-up issue to codify and validate the fix properly.
