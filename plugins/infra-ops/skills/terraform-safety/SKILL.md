---
name: terraform-safety
description: Critical safety rules for Terraform operations to prevent accidental resource destruction. Use before any terraform plan/apply operation, especially when modifying VM or resource definitions.
---

# Terraform Safety

**DANGER**: Terraform operations can accidentally destroy resources if not done carefully!

## Rules for Terraform Changes

### 1. NEVER run `terraform apply` without `-target` when modifying resource definitions

Adding new resources to terraform files can cause existing resources to be destroyed. Always use `-target=module.specific_resource` to apply changes to specific resources only.

### 2. Before ANY terraform apply:

```bash
# ALWAYS run plan first and review carefully
terraform plan -target=module.new_resource

# Check for unexpected destroys - look for red "-" marks
# If you see ANY existing resources being destroyed, STOP
```

### 3. Safe Terraform Workflow

```bash
# Step 1: Add new resource to terraform file

# Step 2: Run plan for ONLY that resource
terraform plan -target=module.new_resource -out=new-resource.tfplan

# Step 3: Review the plan
# Ensure ONLY the new resource is being created
# NO existing resources should show destroy/recreate

# Step 4: Apply ONLY that specific resource
terraform apply new-resource.tfplan
```

### 4. After Terraform Changes, Reconcile State

```bash
# This reconciles state without making changes
terraform plan -refresh-only
terraform apply -refresh-only -auto-approve
```

## What Can Go Wrong

Resources can be accidentally destroyed during `terraform apply` when:
- New resources are added to existing terraform configurations
- Terraform detects a resource was "deleted outside terraform" and removes it from state
- Running apply without `-target` causes all resources to be re-evaluated
- State drift causes Terraform to plan destructive changes

**Lesson**: Always use `-target` when adding new resources to existing terraform configurations!

## Recovery Process

If a resource is accidentally destroyed:
1. Check terraform state: `terraform show | grep <resource-name>`
2. Recreate with: `terraform apply -target=module.<resource-name>`
3. Follow any provisioning documentation to reconfigure
4. Document what happened and how to prevent it

## Pre-Apply Checklist

Before running `terraform apply`:

- [ ] Ran `terraform plan` first
- [ ] Reviewed plan output carefully
- [ ] No unexpected destroy/recreate operations
- [ ] Using `-target` if adding new resources alongside existing ones
- [ ] Saved plan to file with `-out` for review
- [ ] State is in sync (ran `terraform plan -refresh-only` if needed)

## State Management

- **Never** manually edit terraform state files
- Use `terraform state list` to see managed resources
- Use `terraform state show <resource>` to inspect individual resources
- Use `terraform import` to bring existing resources under management
- Use `terraform state rm` only when you understand the implications

## Best Practices

- **Plan before apply, always**
- **Target specific resources** when making changes alongside existing infrastructure
- **Review destroy operations** - understand WHY before allowing them
- **Keep state in sync** with regular refresh-only plans
- **Version control** all terraform configurations
- **Use workspaces** to separate environments (dev, staging, prod)
