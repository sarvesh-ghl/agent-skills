---
name: obsidian-recall
description: Search and retrieve notes from the Obsidian vault by topic, keyword, or tag. Use when the user says "what did I save about X?", "find my notes on X", "search obsidian for X", "pull up my notes on", "what do I have on X?", or wants to find previously saved information in their vault.
---

# Obsidian Recall

Search the Obsidian vault for notes matching a topic, keyword, or tag and present the results to the user. Uses the `user-obsidian` MCP server.

## Workflow

### Step 1: Determine search strategy

From the user's query, determine:
- **Keywords**: Core search terms
- **Scope**: Specific folder or entire vault
- **Type filter**: Looking for a specific note type (session-log, research, guide, bookmark, etc.)

### Step 2: Search

Use a layered search — start broad, narrow if too many results.

**Primary search** — content search:

```
CallMcpTool: user-obsidian / search_notes
{ "query": "{keywords}", "limit": 10, "searchContent": true }
```

**Secondary search** — frontmatter/tags (if the user asks for a specific type):

```
CallMcpTool: user-obsidian / search_notes
{ "query": "{type or tag}", "searchContent": false, "searchFrontmatter": true, "limit": 10 }
```

**Folder-scoped browse** — if the user mentions a project or category:

```
CallMcpTool: user-obsidian / list_directory
{ "path": "{folder}" }
```

### Step 3: Read matching notes

For the most relevant matches (up to 5), read them in batch:

```
CallMcpTool: user-obsidian / read_multiple_notes
{ "paths": ["{match1}", "{match2}", ...], "includeContent": true, "includeFrontmatter": true }
```

For a quick scan without reading full content, use `get_notes_info` instead.

### Step 4: Present results

Format results as a concise summary for the user:

**If 1-3 matches:**
- Show the title, date, and a 2-3 sentence summary of each note
- Include key code snippets or decisions if they're the likely reason for the search
- Offer to read the full note

**If 4+ matches:**
- Show a table of matches with title, date, type, and one-line summary
- Ask which ones the user wants to dive into

**If no matches:**
- Say so clearly
- Suggest related search terms or folders to check

### Result format

```
### Found {N} notes on "{topic}"

| Note | Date | Type | Summary |
|------|------|------|---------|
| [{Title}]({path}) | {date} | {type} | {one-liner} |

Want me to read any of these in full?
```

For detailed results (1-3 matches), present the actual content inline.

## Rules

- **Search before saying "not found".** Try at least 2 search strategies (content + frontmatter, or content + folder browse) before reporting no results.
- **Preserve wikilinks.** When showing note content, keep `[[Note Name]]` links intact so the user can find related notes.
- **Don't dump raw content.** Summarize unless the user explicitly asks for the full note.
- **Mention the file path** so the user can find it in their vault.
