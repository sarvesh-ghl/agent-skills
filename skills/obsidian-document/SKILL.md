---
name: obsidian-document
description: Write structured documentation about a concept, system, process, or architecture and save it to the Obsidian vault. Use when the user says "document this", "write docs on", "explain and save", "write up how this works", or wants a concept/system explained and persisted.
---

# Obsidian Document

Write clear, structured documentation about a concept, system, or process and save it to the Obsidian vault via the `user-obsidian` MCP server.

## Workflow

### Step 1: Identify scope

Determine:
- **Subject**: What to document
- **Audience**: Self-reference, team, or onboarding material
- **Type**: Concept explainer, system overview, process documentation, or architecture doc

If the subject relates to the codebase, explore it first with `SemanticSearch`, `Grep`, and `Read` to ensure accuracy.

### Step 2: Determine vault path

| Subject | Path |
|---------|------|
| Project-specific system/feature | `Projects/{Project Name}/` |
| General concept or technology | `Learning/` |
| Debugging procedure | `Guides/Debugging/` |
| Deployment process | `Guides/Deployment/` |
| Development pattern | `Guides/Development/` |
| Infrastructure topic | `Guides/Infrastructure/` |

**Project mapping:**

| Repository | Path |
|------------|------|
| `automation-workflows-backend` | `Projects/Workflows Backend/` |
| `automation-workflows-frontend` | `Projects/Workflows Frontend/` |
| `spm-ts` | `Projects/SPM/` |
| `automation-workflows-ai` | `Projects/Automation AI/` |

### Step 3: Build the note

```markdown
# {Title}

> **Date:** {YYYY-MM-DD}

---

## Overview

{What this is and why it exists, in 2-3 sentences}

## How It Works

{Core explanation — use diagrams (mermaid), code blocks, and examples}

### {Sub-section}

{Detail}

## Key Concepts

| Concept | Description |
|---------|-------------|
| {Term} | {Definition} |

## Examples

\```{language}
{concrete usage example}
\```

## Gotchas

- {Non-obvious behavior or common mistake}

## Related

- [[{Related Note}]]
- [{External Resource}]({url})
```

Omit empty sections.

### Step 4: Write to Obsidian

```json
{
  "path": "{vault_path}/{Title}.md",
  "content": "{note content}",
  "frontmatter": {
    "tags": ["documentation", "{topic-tags}"],
    "date": "{YYYY-MM-DD}",
    "type": "documentation"
  },
  "mode": "overwrite"
}
```

If a note with the same name exists, use `mode: "append"`.

### Step 5: Confirm

Report the note title, path, and a one-line summary.

## Rules

- **Accuracy first.** If documenting code, read the source before writing. Never guess.
- **Use mermaid diagrams** for flows, state machines, and architecture -- Obsidian renders them natively.
- **Use tables** for term glossaries and comparisons.
- **Include code examples** for technical documentation -- they're the most referenced part.
- **Link related notes** with `[[Note Name]]` wikilinks.
- Keep language direct. Documentation isn't a blog post.
