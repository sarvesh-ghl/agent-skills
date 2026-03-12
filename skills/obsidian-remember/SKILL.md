---
name: obsidian-remember
description: Quickly capture a fact, decision, insight, or piece of context to the Obsidian vault for future reference. Use when the user says "remember this", "note this down", "save this fact", "don't forget", "jot this down", "store this for later", or wants to persist a small piece of information without full documentation.
---

# Obsidian Remember

Quickly capture a small piece of information -- a fact, decision, insight, command, or context -- and save it to the Obsidian vault via the `user-obsidian` MCP server.

This is the lightest-weight save. No ceremony, no structure beyond what's needed.

## Workflow

### Step 1: Classify what to remember

| Type | Example | Path |
|------|---------|------|
| **Decision** | "We chose Redis over Memcached for caching" | `Projects/{Project}/` or `Daily Notes/` |
| **Fact** | "The PubSub topic for agent-node is X" | `Projects/{Project}/` |
| **Command / Config** | "To restart the worker: `yarn worker-dev`" | `Snippets/` |
| **Insight** | "The retry logic silently swallows 429s" | `Projects/{Project}/` or `Bugs & QA/` |
| **Context** | "Client X needs the feature by March 20" | `Daily Notes/` |
| **Misc** | Anything else | `Daily Notes/` |

**Project mapping:**

| Repository | Path |
|------------|------|
| `automation-workflows-backend` | `Projects/Workflows Backend/` |
| `automation-workflows-frontend` | `Projects/Workflows Frontend/` |
| `spm-ts` | `Projects/SPM/` |
| `automation-workflows-ai` | `Projects/Automation AI/` |

### Step 2: Check for existing note

Use `search_notes` to check if a related note already exists. If so, **append** to it instead of creating a new one.

### Step 3: Build the content

**For a new note:**

```markdown
# {Title}

> **Date:** {YYYY-MM-DD}

---

{The thing to remember. Keep it to 1-5 lines.}

{Optional: context on why this matters or where it came from}
```

**For appending to an existing note:**

```markdown

---

### {YYYY-MM-DD}

{The new thing to remember}
```

### Step 4: Write to Obsidian

```json
{
  "path": "{vault_path}/{Title}.md",
  "content": "{content}",
  "frontmatter": {
    "tags": ["remember", "{type: decision|fact|command|insight|context}", "{topic-tag}"],
    "date": "{YYYY-MM-DD}",
    "type": "quick-note"
  },
  "mode": "overwrite"
}
```

Use `mode: "append"` if adding to an existing note.

### Step 5: Confirm

One-line confirmation: what was saved and where.

## Rules

- **Minimal friction.** This should feel instant. Don't over-structure small notes.
- **One note per concept.** If the user says "remember that Redis is on port 6379", that's one note titled "Redis Configuration" -- not a paragraph.
- **Append when related.** If a note on the same concept exists, append to it rather than creating a duplicate.
- **Include the "why".** "We chose X" is less useful than "We chose X because Y".
- For commands and config, always wrap in code blocks.
