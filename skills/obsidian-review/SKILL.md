---
name: obsidian-review
description: Browse and summarize recent activity in the Obsidian vault. Use when the user says "review my recent notes", "what have I been working on?", "show recent sessions", "vault activity", "what did I do this week?", or wants an overview of recent vault entries.
---

# Obsidian Review

Browse recent vault activity and present a summary of what's been saved. Uses the `user-obsidian` MCP server.

## Workflow

### Step 1: Determine scope

From the user's query, determine:

| Query | Scope |
|-------|-------|
| "what have I been working on?" | Recent across all folders |
| "recent sessions" | `Projects/` folders, type: session-log |
| "recent research" | `Learning/` folder |
| "recent meetings" | `Meeting Notes/` folder |
| "what did I do on project X?" | `Projects/{X}/` folder |
| General / unspecified | Recent across all folders |

### Step 2: Gather recent notes

**For vault-wide review:**

```
CallMcpTool: user-obsidian / get_vault_stats
{ "recentCount": 15, "prettyPrint": true }
```

This returns recently modified files across the whole vault.

**For folder-scoped review:**

```
CallMcpTool: user-obsidian / list_directory
{ "path": "{folder}" }
```

Then read frontmatter for the listed files to get dates and types:

```
CallMcpTool: user-obsidian / read_multiple_notes
{ "paths": ["{file1}", "{file2}", ...], "includeContent": false, "includeFrontmatter": true }
```

### Step 3: Read summaries

For the most recent notes (up to 10), batch-read with content:

```
CallMcpTool: user-obsidian / read_multiple_notes
{ "paths": ["{recent1}", "{recent2}", ...], "includeContent": true }
```

### Step 4: Present the review

Format as a chronological activity feed:

```markdown
### Vault Activity — Last {N} days

**{Date}**
- **{Note Title}** ({type}) — {one-line summary}
- **{Note Title}** ({type}) — {one-line summary}

**{Earlier Date}**
- **{Note Title}** ({type}) — {one-line summary}

---

{N} notes across {folders}. Most active: {top folder}.
```

If scoped to a project, include a brief "state of the project" synthesis (what's been done, what's open).

## Rules

- **Group by date** for chronological scanning.
- **Include the note type** (session-log, research, guide, etc.) so the user can see the mix of activity.
- **Summarize, don't dump.** One line per note unless the user asks for more.
- **Call out open items.** If recent notes have `## Open Items` or `- [ ]` tasks, surface them — that's likely the most actionable part.
- **Mention note paths** so the user can navigate in Obsidian.
