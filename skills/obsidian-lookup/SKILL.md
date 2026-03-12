---
name: obsidian-lookup
description: Quick check whether relevant notes already exist in the Obsidian vault before starting work. Use when the user says "check if I have notes on X", "did I already document X?", "do I have anything on X?", "have I saved anything about X?", or before researching/documenting a topic to avoid duplicating prior work.
---

# Obsidian Lookup

Quick pre-check to see if the vault already contains notes on a topic before starting new work. Prevents re-research, re-documentation, and duplicate notes. Uses the `user-obsidian` MCP server.

## When to Use

This skill is lighter than `obsidian-recall`. Use it when the goal is a yes/no answer with brief context, not a deep retrieval. Ideal as a first step before using write skills like `obsidian-document`, `obsidian-research`, or `obsidian-guide`.

## Workflow

### Step 1: Quick search

Run a content search with a modest limit:

```
CallMcpTool: user-obsidian / search_notes
{ "query": "{topic keywords}", "limit": 5, "searchContent": true }
```

### Step 2: Assess results

**If matches found** — read frontmatter only to get dates and types:

```
CallMcpTool: user-obsidian / read_multiple_notes
{ "paths": ["{match1}", "{match2}", ...], "includeContent": false, "includeFrontmatter": true }
```

**If no matches** — try a broader search or check the likely folder:

```
CallMcpTool: user-obsidian / list_directory
{ "path": "{likely_folder}" }
```

### Step 3: Report

**If notes exist:**

```
Found {N} existing notes on "{topic}":

1. **{Title}** ({date}, {type}) — in `{path}`
2. **{Title}** ({date}, {type}) — in `{path}`

Want me to read any of these, or proceed with creating new content?
```

**If nothing found:**

```
No existing notes on "{topic}" in the vault. Safe to proceed.
```

**If partial match** (related but not exactly the same topic):

```
No direct match for "{topic}", but found related notes:

1. **{Title}** — `{path}` (related: {why})

Want me to check these first, or start fresh?
```

## Rules

- **Speed over depth.** This is a quick check — don't read full note content unless asked.
- **Report and ask** before proceeding. Let the user decide whether to build on existing notes or start fresh.
- **Suggest appending** if a closely related note exists, rather than creating a duplicate.
