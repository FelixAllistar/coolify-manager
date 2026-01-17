---
name: coolify-manager
description: Manage self-hosted infrastructure via Coolify. Use this to deploy apps, debug build failures, manage databases, and execute commands inside remote Docker containers.
---

# Coolify Infrastructure Manager

This skill allows you to manage a self-hosted Coolify instance. It handles deployment lifecycles, resource configuration, and advanced debugging via SSH.

## 🧠 Decision Tree

**Step 1: Analyze the User Request.**
Determine which of the following categories the request falls into and follow the instructions.

### A. "Fix it" / "Debug it" (Operations)
*User asks: "Why did the build fail?", "Show me logs", "Restart the service"*
1. **Check Status:** Run `coolify app get <uuid>` to see if it's running or exited.
2. **Get Logs:** - If build failed: `coolify app deployments logs <app_uuid>` (Add `--debuglogs` if unclear).
   - If runtime error: `coolify app logs <uuid>`.
3. **Action:** If stuck, `coolify app restart <uuid>`.

### B. "Change it" (Configuration)
*User asks: "Update environment variables", "Change the domain", "Add a database"*
1. **Consult Reference:** Read `resources/common-workflows.md` for specific command patterns.
2. **Environment Vars:** Remember that `coolify app env sync` is safer than adding one by one if the user provides a list.

### C. "Run command inside container" (Advanced Exec)
*User asks: "Run rails console", "Check /tmp in the container", "Run database migration manually"*
1. **Context:** Coolify CLI does not natively support `exec`.
2. **Solution:** You MUST use the `scripts/container-exec.sh` helper script provided in this skill.
3. **Method:** - First, find the `app_uuid`.
   - Run: `./scripts/container-exec.sh <app_uuid> "<command>"`

## ⚡ Best Practices

1. **Safety First:**
   - Never use `--force` on `delete` commands unless the user explicitly requested it.
   - Always verify the context (`coolify context verify`) before running destructive commands to ensure you aren't in `prod` when you think you are in `staging`.

2. **Resource Discovery:**
   - If you don't know the UUID, search by listing resources: `coolify resources list`.
   - Always prefer UUIDs over names for critical operations to avoid ambiguity.

3. **Handling "Impossible" Actions:**
   - If the CLI lacks a feature (like direct file upload or shell access), refer to the **SSH Tunneling** protocol in `resources/common-workflows.md` or use the script in `scripts/`.