---
name: obsidian-digest
description: Synthesize multiple Obsidian vault notes on a topic into a coherent summary or briefing. Use when the user says "summarize my notes on X", "give me a digest of X", "what do I know about X?", "brief me on X from my notes", "compile my notes on X", or wants a synthesized view across several related notes.
---

# Obsidian Digest

Read multiple related notes from the Obsidian vault and synthesize them into a coherent summary. Uses the `user-obsidian` MCP server.

## Workflow

### Step 1: Identify scope

From the user's query, determine:
- **Topic**: What to synthesize across
- **Scope**: Entire vault, specific project, specific folder, or specific note type

### Step 2: Gather notes

**Search by topic:**

```
CallMcpTool: user-obsidian / search_notes
{ "query": "{topic}", "limit": 15, "searchContent": true }
```

**Or browse a project folder:**

```
CallMcpTool: user-obsidian / list_directory
{ "path": "{Projects/{Name}|Learning|Guides/{Category}}" }
```

**Filter by type** if the user wants a specific kind (e.g., "summarize my session logs"):

```
CallMcpTool: user-obsidian / search_notes
{ "query": "{type-tag}", "searchFrontmatter": true, "limit": 15 }
```

### Step 3: Read the notes

Batch-read the relevant notes (up to 10 at a time):

```
CallMcpTool: user-obsidian / read_multiple_notes
{ "paths": ["{note1}", "{note2}", ...], "includeContent": true, "includeFrontmatter": true }
```

If more than 10 notes match, prioritize by:
1. Most recent first
2. Most relevant to the query
3. Notes with open items or action items

### Step 4: Synthesize

Build a digest that **connects** the notes, not just lists them. Structure:

```markdown
## Digest: {Topic}

> Synthesized from {N} notes ({date range})

### State of Things

{2-3 paragraph narrative summarizing the overall picture — what's been done, what's known, what's open}

### Timeline

| Date | Note | Key Point |
|------|------|-----------|
| {date} | {title} | {most important takeaway} |

### Key Decisions

- **{Decision}** ({date}) — {rationale}

### Patterns & Recurring Themes

- {Theme that appears across multiple notes}

### Open Items

- [ ] {Aggregated from across notes}
- [ ] {Aggregated from across notes}

### Source Notes

- `{path1}` — {title}
- `{path2}` — {title}
```

Omit empty sections.

### Step 5: Optionally save the digest

If the user wants to persist the digest, save it using `write_note`:

```json
{
  "path": "{appropriate_folder}/Digest - {Topic}.md",
  "content": "{digest content}",
  "frontmatter": {
    "tags": ["digest", "{topic-tags}"],
    "date": "{YYYY-MM-DD}",
    "type": "digest",
    "source-notes": ["{path1}", "{path2}"]
  },
  "mode": "overwrite"
}
```

Only save if the user asks to persist it. Otherwise, just present inline.

## Rules

- **Synthesize, don't concatenate.** The value of a digest is the connections and narrative, not a list of summaries.
- **Aggregate open items.** Collect all `- [ ]` tasks from source notes into one list — this is usually the most actionable output.
- **Highlight contradictions.** If notes disagree or decisions changed over time, call it out.
- **Cite source notes** so the user can drill into any specific note.
- **Chronological timeline** helps the user see how understanding evolved.
- **Ask before saving.** Present the digest inline first; only write to vault if requested.
