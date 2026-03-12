---
name: obsidian-research
description: Research a topic using web search and codebase exploration, then save structured findings to the Obsidian vault. Use when the user says "research and save", "research this", "look into this and save to obsidian", "investigate and document", or wants to explore a topic and persist the findings.
---

# Obsidian Research

Research a topic thoroughly (web, codebase, or both), then save structured findings to the Obsidian vault via the `user-obsidian` MCP server.

## Workflow

### Step 1: Clarify scope

Determine from conversation context:
- **Topic**: What to research
- **Depth**: Quick overview vs. deep dive
- **Sources**: Web search, codebase exploration, or both
- **Angle**: Why the user cares (evaluating a tool? understanding a concept? comparing options?)

### Step 2: Conduct research

Use the appropriate tools:
- **Web**: `WebSearch` and `WebFetch` for external information
- **Codebase**: `SemanticSearch`, `Grep`, `Read` for internal patterns
- **Both**: Cross-reference external best practices with current implementation

Collect:
- Key findings and facts
- Comparisons or trade-offs (if applicable)
- Code examples or patterns
- Sources and links

### Step 3: Determine vault path

| Topic | Path |
|-------|------|
| Technology / concept / library | `Learning/` |
| Project-specific research | `Projects/{Project Name}/` |
| General | `Learning/` |

**Project mapping** (same as session-log):

| Repository | Path |
|------------|------|
| `automation-workflows-backend` | `Projects/Workflows Backend/` |
| `automation-workflows-frontend` | `Projects/Workflows Frontend/` |
| `spm-ts` | `Projects/SPM/` |
| `automation-workflows-ai` | `Projects/Automation AI/` |

### Step 4: Build the note

```markdown
# {Title}

> **Date:** {YYYY-MM-DD}
> **Topic:** {topic}

---

## TL;DR

{2-3 sentence summary of the key takeaway}

## Findings

### {Finding 1}

{Detail with supporting evidence}

### {Finding 2}

{Detail with supporting evidence}

## Comparisons

| Criteria | Option A | Option B |
|----------|----------|----------|
| ... | ... | ... |

## Code Examples

\```{language}
{relevant code}
\```

## Sources

- [{title}]({url})
- [{title}]({url})

## Takeaways

- {Actionable conclusion 1}
- {Actionable conclusion 2}
```

Omit any section that has no content.

### Step 5: Write to Obsidian

```json
{
  "path": "{vault_path}/{Title}.md",
  "content": "{note content}",
  "frontmatter": {
    "tags": ["research", "{topic-tags}"],
    "date": "{YYYY-MM-DD}",
    "type": "research",
    "status": "completed"
  },
  "mode": "overwrite"
}
```

### Step 6: Confirm

Report the note title, path, and the TL;DR.

## Rules

- **Use tables** for comparisons -- they match the vault style and are scannable.
- **Cite sources** with links. Future-you will want to know where info came from.
- **Include code** when the research is technical. Examples are worth more than explanations.
- **Be opinionated** in the Takeaways section -- state what you'd recommend, not just what exists.
- Link related vault notes with `[[Note Name]]` wikilinks.
