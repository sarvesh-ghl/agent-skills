---
name: qa-bug-fix-workflow
description: End-to-end workflow for fixing QA bugs from ClickUp cards. Use when the user shares a ClickUp task starting with '[QA]', mentions a QA card, asks to fix QA bugs, or references a ClickUp bug-fix task URL. Handles sub-task triage, multi-repo branch management, code fixes, staging deployment via Jenkins, build monitoring, and ClickUp status updates.
license: Apache-2.0
metadata:
  author: sarvesh-ghl
  version: "1.0"
compatibility: Requires ClickUp MCP, Jenkins MCP (git-jenkins), and gh CLI. Workspace should contain all relevant repos.
---

# QA Bug-Fix Workflow

Structured workflow for triaging, fixing, deploying, and closing QA bug sub-tasks from a ClickUp parent card. Supports multi-repo workspaces where a single feature spans backend, frontend, and other services.

## Prerequisites

- ClickUp MCP connected (for task read/write)
- Jenkins MCP connected (for staging deployments)
- Git repos for the feature checked out in the workspace
- `yarn` available for build/lint commands

## Workflow Checklist

Copy and track progress with TodoWrite:

```
- [ ] Step 1: Load QA card and sub-tasks
- [ ] Step 2: Branch setup and master merge
- [ ] Step 3: Fix bugs (one by one)
- [ ] Step 4: Build, lint, and commit
- [ ] Step 5: Identify and merge into staging branch(es)
- [ ] Step 6: Deploy to staging via Jenkins
- [ ] Step 7: Monitor builds and update ClickUp
- [ ] Step 8: Close out parent card
```

## Workflow

### Step 1: Load QA card and sub-tasks

1. Fetch the parent task using `clickup_get_task` with `subtasks: true`.
2. Present a summary table of all sub-tasks: name, status, assignee.
3. Filter to sub-tasks assigned to the current user (use `clickup_find_member_by_name` with "me" if needed).
4. Confirm the filtered list with the user before proceeding.

### Step 2: Branch setup and master merge

The workspace may contain **multiple repos** for a single feature.

1. **Identify repos**: List all git repos in the workspace (`git status` in each root).
2. **Find the feature branch**: Check the current branch in each repo. Look for a common feature branch name (e.g., `feat/some-feature`). If unclear, **ask the user**.
3. **Checkout and sync** (for each relevant repo):
   - `git checkout <feature-branch>`
   - `git fetch origin`
   - Determine the default branch (`master` or `main`): `git remote show origin | head -5`
   - `git merge origin/<default-branch>`
   - If conflicts arise, search for conflict markers (`<<<<<<<`) with Grep, resolve with StrReplace.
   - If code conflicts are non-trivial, present them to the user for guidance.

### Step 3: Fix bugs one by one

For each sub-task (in order):

1. **Mark in-progress**: `clickup_update_task` → status: `in dev sprint`
2. **Understand the bug**: Read the sub-task name, description, and any comments (`clickup_get_task_comments`).
3. **Investigate**: Search the codebase for relevant files using the bug description as context.
4. **Fix**: Make the code changes across whichever repo(s) are affected.
5. **Move to next** sub-task.

> Do NOT commit yet — batch all fixes, then build/lint/commit in Step 4.

### Step 4: Build, lint, and commit

After all bugs are fixed:

1. **Build** each affected repo: `yarn build` (or the repo's build command).
2. **Fix build errors** if any.
3. **Lint**: `yarn lint:fix` or equivalent.
4. **Commit** with a descriptive message referencing the QA card name.
   - If pre-commit hooks fail (typically lint errors on changed files), fix and retry.
   - Do NOT use `--no-verify`.
5. **Push** the feature branch to remote in each repo.

### Step 5: Identify and merge into staging branch(es)

For each affected repo:

1. **Check for `jenkins-jobs.json`** in the repo root. If found, read the `stagingBranch` field.
2. **Find the latest staging branch**:
   - `git fetch --all`
   - List remote staging branches and find the highest version (format: `staging-vNN`).
   - Compare with `jenkins-jobs.json` value. Use the higher one.
   - If unable to determine, **ask the user**.
3. **Merge feature → staging**:
   - `git checkout <staging-branch> && git pull origin <staging-branch>`
   - `git merge <feature-branch>`
   - Resolve conflicts (same approach as Step 2).
4. **Build**: `yarn build` to verify.
5. **Push**: `git push origin <staging-branch>`
6. **Return**: `git checkout <feature-branch>`

### Step 6: Deploy to staging via Jenkins

For each repo with a `jenkins-jobs.json`:

1. Read the job configuration (app names, job paths, parameters).
2. If the repo is NOT a monorepo (no `apps/` dir), create a temp one:
   ```bash
   mkdir -p apps/<app-name>
   ```
3. Trigger deploy using `deploy_to_staging`:
   - `merge: false` (already merged manually)
   - `type`: inferred from config (server, frontend, workers)
   - Enable relevant worker params based on the feature
4. Clean up the temp `apps/` dir after triggering.

### Step 7: Monitor builds and update ClickUp

1. **Poll build status** using `check_build_status` with the pipeline path from `jenkins-jobs.json`. Check every 30–60 seconds.
2. If the build **fails**:
   - Fetch logs with `get_build_log`.
   - Diagnose and fix the issue, then re-deploy (return to Step 6).
3. If the build **succeeds**:
   - For each fixed sub-task:
     - `clickup_create_task_comment` — what was fixed, which staging branch, which repo.
     - `clickup_update_task` → status: `ready for qa`

### Step 8: Close out parent card

Once **all** sub-tasks are in `ready for qa`:

1. `clickup_update_task` on the parent → status: `ready for qa`.
2. `clickup_create_task_comment` on the parent with a summary of all fixes and deployment details.

## Guidelines

### When to ask the user

| Situation | Action |
|-----------|--------|
| Can't find feature branch | Ask user |
| Can't determine staging branch | Ask user |
| Non-trivial code merge conflicts | Present to user |
| Sub-task description is empty | Use task name to infer; confirm approach with user |

### Conflict resolution

- **Docs-only conflicts**: Auto-resolve by combining both sides.
- **Code conflicts**: Attempt auto-resolution for simple cases (adjacent additions). Present to user if overlapping logic changes.

### Commit and deploy

- Never use `--no-verify` on commits.
- Always build before pushing to staging.
- If Jenkins MCP requires `apps/` directory, create temp dir → deploy → clean up.
- Set `merge: false` on `deploy_to_staging` since merging is done manually.
