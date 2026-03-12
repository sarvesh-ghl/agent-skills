---
name: obsidian-bookmark
description: Save bookmarks, links, snippets, or quick ideas to the Obsidian vault. Use when the user says "save this", "bookmark this", "save this link", "save this snippet", "clip this", or wants to quickly capture something for later reference.
---

# Obsidian Bookmark

Quickly save a link, code snippet, idea, or reference to the Obsidian vault via the `user-obsidian` MCP server.

## Workflow

### Step 1: Identify what to save

Determine the type of bookmark from conversation context:

| Type | Content | Vault Path |
|------|---------|------------|
| **Link** | URL + title + optional notes | `Bookmarks/` |
| **Code snippet** | Reusable code pattern | `Snippets/` |
| **Idea / Quick note** | Freeform thought or idea | `Bookmarks/` |

Extract:
- **Title**: Short descriptive name
- **Content**: The thing being saved (URL, code, text)
- **Context**: Why it's worth saving (1 sentence)
- **Source**: Where it came from (conversation, web, codebase)

### Step 2: Build the note

**For links:**

```markdown
# {Title}

> {One-line description of what this is and why it's useful}

**URL:** {url}

## Notes

{Any additional context, key takeaways, or why this was bookmarked}
```

**For code snippets:**

```markdown
# {Title}

> {What this snippet does}

## Code

\```{language}
{code}
\```

## Usage

{When/where to use this pattern}
```

**For ideas / quick notes:**

```markdown
# {Title}

{The idea or note content}

## Context

{Where this came from or why it matters}
```

### Step 3: Write to Obsidian

Use `CallMcpTool` with server `user-obsidian`, tool `write_note`:

```json
{
  "path": "{Bookmarks|Snippets}/{Title}.md",
  "content": "{note content}",
  "frontmatter": {
    "tags": ["bookmark", "{type: link|snippet|idea}", "{topic-tag}"],
    "date": "{YYYY-MM-DD}",
    "type": "bookmark",
    "source": "{conversation|web|codebase}"
  },
  "mode": "overwrite"
}
```

If a note with the same name exists, use `mode: "append"`.

### Step 4: Confirm

Report the note title, path, and a one-line summary.

## Rules

- Keep it short. Bookmarks are reference material, not essays.
- For code snippets, always include the language tag and a one-liner on when to use it.
- Add topic tags (e.g., `typescript`, `kubernetes`, `mongodb`) for discoverability.
- Use `[[Note Name]]` wikilinks if the bookmark relates to an existing vault note.
