---
name: obsidian-guide
description: Write a step-by-step guide or tutorial and save it to the Obsidian vault. Use when the user says "write a guide", "create a tutorial", "how-to guide", "write steps for", "document how to", or wants a procedural walkthrough saved for future reference.
---

# Obsidian Guide

Write a practical, step-by-step guide or tutorial and save it to the Obsidian vault via the `user-obsidian` MCP server.

## Workflow

### Step 1: Scope the guide

Determine:
- **Task**: What the guide teaches someone to do
- **Prerequisites**: What the reader needs before starting
- **Complexity**: Quick reference (5-10 steps) vs. comprehensive tutorial

If the guide covers codebase procedures, explore the code first with `SemanticSearch`, `Grep`, and `Read`.

If the topic needs external info, use `WebSearch`.

### Step 2: Determine vault path

Map to `Guides/` subcategory:

| Topic | Path |
|-------|------|
| Debugging procedures | `Guides/Debugging/` |
| Deployment processes | `Guides/Deployment/` |
| Development workflows | `Guides/Development/` |
| Infrastructure/DevOps | `Guides/Infrastructure/` |
| Doesn't fit a subcategory | `Guides/` |

### Step 3: Build the guide

```markdown
# {Title}

> **Date:** {YYYY-MM-DD}

---

## Prerequisites

- {Requirement 1}
- {Requirement 2}

## Steps

### 1. {First step title}

{What to do and why}

\```bash
{command or code}
\```

### 2. {Second step title}

{What to do and why}

### 3. {Continue as needed}

{...}

## Verification

How to confirm the procedure worked:

\```bash
{verification command}
\```

Expected output: {what success looks like}

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| {symptom} | {root cause} | {solution} |

## Related

- [[{Related Guide}]]
```

Omit empty sections.

### Step 4: Write to Obsidian

```json
{
  "path": "{vault_path}/{Title}.md",
  "content": "{guide content}",
  "frontmatter": {
    "tags": ["guide", "{category}", "{topic-tags}"],
    "date": "{YYYY-MM-DD}",
    "type": "guide"
  },
  "mode": "overwrite"
}
```

### Step 5: Confirm

Report the note title, path, and number of steps.

## Rules

- **Every step must be actionable.** Start with a verb. "Configure X", "Run Y", "Verify Z".
- **Include the actual commands and code.** Don't say "run the migration" -- show the exact command.
- **Add a Verification section.** The reader should be able to confirm they did it right.
- **Add a Troubleshooting table** for known failure modes.
- **Keep steps atomic.** One thing per step. If a step has sub-steps, break it up.
- Link related guides with `[[Note Name]]` wikilinks.
