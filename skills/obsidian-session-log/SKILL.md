---
name: obsidian-session-log
description: Document the current Cursor conversation as a structured note in Obsidian via the obsidian MCP. Use when the user says "save to obsidian", "log this session", "document this conversation", or invokes this skill at the end of a coding session.
---

# Obsidian Session Log

Save a structured summary of the current Cursor conversation to the Obsidian vault using the `user-obsidian` MCP server.

## Workflow

### Step 1: Analyze the conversation

Review the full conversation and extract:

- **Title**: A concise descriptive title (e.g. "FlowGuard Rate Limiting Refactor", "Fix Contact Search 500 Error")
- **Project(s)**: Which repo/project was worked on
- **Category**: `feature` | `bugfix` | `refactor` | `debug` | `research` | `config` | `docs`
- **Summary**: 2-3 sentence overview of what was accomplished
- **Changes**: List of concrete changes made (with file paths)
- **Key decisions**: Architectural or design decisions made and why
- **Bugs found/fixed**: Any bugs encountered with root cause and fix (if applicable)
- **Snippets**: Reusable code patterns or commands worth preserving (if any)
- **Open items**: Anything left unfinished or flagged for follow-up

### Step 2: Determine the Obsidian path

Map the project to its vault folder:

| Repository / Context | Obsidian Path |
|---|---|
| `automation-workflows-backend` | `Projects/Workflows Backend/` |
| `automation-workflows-frontend` | `Projects/Workflows Frontend/` |
| `spm-ts` | `Projects/SPM/` |
| `automation-workflows-ai` | `Projects/Automation AI/` |
| Multiple projects | Use the primary project; mention others in tags |
| General / no specific project | `Daily Notes/` |

**Filename**: `{Title}.md` (title-cased, spaces allowed, no special chars except hyphens)

If a note with the same name already exists, append to it using `mode: "append"` instead of overwriting.

### Step 3: Build the note

Use this template — omit any section that has no content:

```markdown
# {Title}

> **Date:** {YYYY-MM-DD}
> **Project:** {repo name}
> **Category:** {category}

---

## Summary

{2-3 sentence overview}

## Changes Made

- `path/to/file.ts` — {what changed and why}
- `path/to/other.ts` — {what changed and why}

## Key Decisions

- **{Decision}**: {Rationale}

## Bugs Found & Fixed

### {Bug title}

**Error:** `{error message}`
**Root Cause:** {explanation}
**Fix:**
\```typescript
// corrected code
\```

## Snippets

\```typescript
// reusable pattern worth remembering
\```

## Open Items

- [ ] {follow-up task}
- [ ] {thing to revisit}
```

### Step 4: Write to Obsidian

Use `CallMcpTool` with server `user-obsidian`, tool `write_note`:

```json
{
  "path": "{obsidian_path}/{Title}.md",
  "content": "{note content from step 3}",
  "frontmatter": {
    "tags": ["{category}", "cursor-session", "project/{project-slug}"],
    "date": "{YYYY-MM-DD}",
    "project": "{repo-name}",
    "type": "session-log",
    "status": "completed"
  },
  "mode": "overwrite"
}
```

**Tag conventions:**
- Always include `cursor-session` and `project/{slug}` (e.g. `project/workflows-backend`)
- Add the category tag (e.g. `bugfix`, `feature`)
- Add technology tags for key tools/libs involved (e.g. `langchain`, `pubsub`, `mongodb`)

### Step 5: Confirm

After saving, report back:
- The note title and path
- A one-line summary of what was logged

## Rules

- **Be concise.** This is a reference note, not a transcript. Distill the session to its essence.
- **Preserve code.** Include actual code snippets for bugs fixed and patterns discovered — these are the most valuable parts for future reference.
- **Use tables** for comparisons, parameter mappings, and before/after contrasts (matches existing vault style).
- **Link related notes** with `[[Note Name]]` wikilinks if you know a related note exists in the vault.
- **Skip boilerplate.** If a section would only say "none" or "n/a", omit it entirely.
- If the session was purely exploratory with no code changes, use category `research` and focus the note on findings and conclusions.
