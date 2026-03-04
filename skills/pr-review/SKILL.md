---
name: pr-review
description: Review GitHub pull requests with inline code comments. Use when the user provides a PR link, asks to review a PR, mentions "review this PR", "check this pull request", or provides a GitHub PR URL. Fetches PR details via gh CLI, analyzes changes, and posts inline review comments after user approval.
license: Apache-2.0
metadata:
  author: sarvesh-ghl
  version: "1.0"
compatibility: Requires gh CLI authenticated with repo access
---

# PR Review

Interactive GitHub PR review workflow: fetch PR, analyze changes, present findings, iterate with user, and post the review with inline comments.

## Prerequisites

- `gh` CLI authenticated (`gh auth status`)
- Repository access for the target PR

## Workflow

### Step 1: Parse the PR URL

Extract `owner`, `repo`, and `pr_number` from the URL.

Supported formats:
- `https://github.com/owner/repo/pull/123`
- `owner/repo#123`
- `#123` (uses current repo context)

### Step 2: Fetch PR details

Run these commands to gather context:

```bash
# PR metadata
gh pr view <PR_URL> --json title,body,author,baseRefName,headRefName,files,additions,deletions,changedFiles,labels,comments,reviews

# Full diff
gh pr diff <PR_URL>

# Individual changed files (for reading full file context when needed)
gh pr view <PR_URL> --json files --jq '.files[].path'
```

Present a brief summary to the user:
- PR title and author
- Base <- Head branch
- Number of files changed, additions, deletions
- PR description (truncated if long)

### Step 3: Ask review style

Before reviewing, ask the user which review depth they want:

| Style | Focus |
|-------|-------|
| **Strict** | All issues: bugs, security, logic, style, naming, formatting |
| **Balanced** | Bugs, security, logic errors; style only if significant |
| **Critical only** | Only bugs, security vulnerabilities, and breaking changes |

Use the AskQuestion tool with these three options.

### Step 4: Analyze the diff

Review the diff file-by-file. For each changed file:

1. Read the full diff hunks
2. If needed, fetch the full file for surrounding context using `gh api repos/{owner}/{repo}/contents/{path}?ref={head_branch}`
3. Identify issues based on the chosen review style

For each finding, record:
- **File path** (relative to repo root)
- **Line number** in the diff (the new-file line number)
- **Severity**: 🔴 Critical | 🟡 Suggestion | 🟢 Nitpick
- **Category**: bug, security, performance, logic, style, naming, documentation
- **Description**: What's wrong and why
- **Suggestion**: How to fix it (with code snippet if applicable)

### Step 5: Present findings to user

Format the review as a structured report:

```
## PR Review: <title>

**Files reviewed**: X | **Findings**: Y (🔴 A critical, 🟡 B suggestions, 🟢 C nitpicks)

---

### file/path.ts

**Line 42** 🔴 **Bug** — Null reference possible
`user.profile.name` accessed without null check. Will throw if `profile` is undefined.
```suggestion
const name = user.profile?.name ?? 'Unknown'
```

**Line 87** 🟡 **Performance** — N+1 query in loop
Database query inside `forEach` causes N+1 problem. Batch the IDs and query once.

---

### another/file.ts

**Line 15** 🟢 **Naming** — Unclear variable name
`d` should be `durationMs` for clarity.

---

## Summary
[1-2 sentence overall assessment]

## Recommended action: [APPROVE / REQUEST_CHANGES / COMMENT]
[Brief justification for the recommendation]
```

### Step 6: Get user approval

Ask the user:

| Option | Description |
|--------|-------------|
| **Post as-is** | Submit the review to GitHub exactly as shown |
| **Edit findings** | User provides changes to the review (add, remove, or modify findings) |
| **Change action** | Change the recommended review action (APPROVE/REQUEST_CHANGES/COMMENT) |
| **Cancel** | Discard the review |

If the user chooses "Edit findings":
1. Apply the user's requested changes
2. Present the updated review
3. Ask for approval again
4. Repeat until the user is satisfied

### Step 7: Post the review

Use the GitHub API to create a review with inline comments:

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/reviews \
  --method POST \
  --field event='{ACTION}' \
  --field body='{SUMMARY_BODY}' \
  --field comments='[
    {
      "path": "file/path.ts",
      "line": 42,
      "body": "🔴 **Bug** — Null reference possible\n\n`user.profile.name` accessed without null check.\n\n```suggestion\nconst name = user.profile?.name ?? '\''Unknown'\''\n```"
    }
  ]'
```

**Mapping severity to review action:**
- Any 🔴 Critical findings -> recommend `REQUEST_CHANGES`
- Only 🟡 and 🟢 findings -> recommend `COMMENT`
- No findings -> recommend `APPROVE`

Always confirm the action with the user before posting.

After posting, provide the review URL back to the user.

## Review Guidelines

### What to look for

- **Bugs**: Null derefs, off-by-one, race conditions, unhandled errors
- **Security**: Injection, auth bypass, sensitive data exposure, insecure defaults
- **Logic**: Wrong conditions, missing edge cases, incorrect control flow
- **Performance**: N+1 queries, unnecessary allocations, missing indexes
- **API design**: Breaking changes, missing validation, inconsistent patterns
- **Style** (if strict mode): Naming, formatting, dead code, missing types

### What to skip

- Auto-generated files (lock files, snapshots, migrations unless hand-edited)
- Formatting-only changes already handled by a formatter
- Dependency version bumps (unless security-relevant)
- Changes outside the diff context (only review what's in the PR)

### Comment quality

- Be specific: point to the exact line and explain *why* it's an issue
- Provide fixes: include code suggestions when possible
- Be respectful: frame as improvements, not criticisms
- Group related: if the same pattern repeats, note it once and say "same pattern at lines X, Y, Z"
