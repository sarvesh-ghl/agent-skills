---
name: pr-review
description: Review GitHub pull requests with inline code comments. Use when the user provides a PR link, asks to review a PR, mentions "review this PR", "check this pull request", or provides a GitHub PR URL. Fetches PR details via gh CLI, analyzes changes, and posts inline review comments after user approval.
license: Apache-2.0
metadata:
  author: sarvesh-ghl
  version: "1.1"
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

### Step 5: Present findings to user (internal preview)

This format is for the USER to review in chat. It is structured for easy scanning.

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

## Summary
[1-2 sentence overall assessment]

## Recommended action: [APPROVE / REQUEST_CHANGES / COMMENT]
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

### Step 7: Rewrite for GitHub (CRITICAL)

**The review is posted under the user's GitHub account. It MUST read like the user wrote it themselves.** Before posting, rewrite every comment and the summary body using these rules:

#### Voice & tone rules

| Rule | Bad (AI voice) | Good (human voice) |
|------|----------------|---------------------|
| First person | "The reviewer notes that..." | "I noticed that..." |
| No severity labels | "🟡 **Logic** — Fragile assumption" | "This assumption looks fragile —" |
| No emoji prefixes | "🔴 **Bug** — Null ref" | "This will throw if `profile` is undefined." |
| No category tags | "**Performance** — N+1 query" | "This is doing a query per iteration — worth batching." |
| No stats headers | "**Files reviewed**: 5 \| **Findings**: 4 (🔴 0...)" | "Looks good overall, a few things I'd tighten up:" |
| No "Recommended action" | "**Recommended action**: COMMENT" | *(just omit — the GitHub action speaks for itself)* |
| Conversational | "Consider identifying the root by..." | "What if we found the root by checking..." |
| Direct questions | "This warrants a clarifying comment." | "Could we add a comment here explaining why...?" |
| Contractions | "This does not handle..." | "This doesn't handle..." |
| Casual where natural | "The function itself lacks unit tests for its actual behavior" | "I think this could use its own unit test —" |

#### Summary body (the top-level review comment)

Write 2-4 sentences as if you're leaving a quick note on a colleague's PR. Examples:

```
Nice fix — the clear-then-rebuild approach is solid for reconciling parentKeys
after moves. Left a few suggestions, nothing blocking.
```

```
Looks good! I had a couple of questions about the edge cases in the
merge logic, and a minor nit on naming. Happy to approve once those
are addressed.
```

**Never** include: file counts, finding counts, severity tallies, "Recommended action", bullet-point stat lines, or structured headers in the posted summary.

#### Inline comments

Each inline comment should read like a human typed it into the GitHub review box:

**Bad** (AI-generated):
```
🟡 **Logic** — Fragile root node assumption

`templates[0].parentKey = undefined` assumes the first array element is always
the root node. If a move operation or array mutation ever places a non-root at
index 0, this silently corrupts the graph by clearing a valid `parentKey`.

Consider identifying the root by the absence of any `next` pointer referencing
it rather than relying on array position:
```

**Good** (human):
```
`templates[0]` assumes the first element is always root — if a move ever
shuffles the array, this would silently clear a valid parentKey.

What about finding the root by which node nobody points to?
```

Rules for inline comments:
- No severity emoji or bold category prefix
- Start directly with the observation or question
- Use "I", "we", contractions, and questions naturally
- Keep it concise — say it in 2-3 sentences if possible, not a paragraph
- Code suggestions are fine and encouraged (use GitHub's ` ```suggestion ` syntax)
- If referencing another reviewer's comment, say "I agree with X" or "building on X's point" — never "the reviewer's concern"

### Step 8: Post the review

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
      "body": "This will throw if `profile` is undefined.\n\n```suggestion\nconst name = user.profile?.name ?? '\''Unknown'\''\n```"
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
