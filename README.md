# Agent Skills

[![Skills](https://img.shields.io/badge/skills.sh-leaderboard-black)](https://skills.sh)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)

Reusable capabilities for AI coding agents. Each skill teaches an agent how to perform a specific task — reviewing PRs, generating docs, automating workflows — in a repeatable, high-quality way.

Compatible with **Cursor**, **Claude Code**, **Windsurf**, **Cline**, **Codex**, and [any agent that supports the Agent Skills spec](https://agentskills.io/specification).

## Quick Install

```bash
# Install all skills
npx skills add sarvesh-ghl/agent-skills

# Install a specific skill
npx skills add sarvesh-ghl/agent-skills --skill pr-review

# Or use the full URL path
npx skills add https://github.com/sarvesh-ghl/agent-skills/tree/main/skills/pr-review
```

## Skills

| Skill | Description |
|-------|-------------|
| [`pr-review`](skills/pr-review/) | Review GitHub PRs with inline code comments via `gh` CLI. Fetches diff, analyzes changes, presents findings, iterates with you, and posts the review. |
| [`qa-bug-fix-workflow`](skills/qa-bug-fix-workflow/) | End-to-end QA bug-fix workflow from ClickUp cards. Triages sub-tasks, manages multi-repo branches, fixes bugs, deploys to staging via Jenkins, and updates ClickUp statuses. |

### Obsidian Vault Management

Skills for managing an Obsidian vault directly from AI agent conversations. Requires the [Obsidian MCP server](https://github.com/MarkusPfworlds/obsidian-mcp).

**Write skills** — capture information into the vault:

| Skill | Triggers | Description |
|-------|----------|-------------|
| [`obsidian-session-log`](skills/obsidian-session-log/) | "save to obsidian", "log this session" | Document coding sessions with changes, decisions, bugs, and open items. |
| [`obsidian-bookmark`](skills/obsidian-bookmark/) | "save this", "bookmark this", "clip this" | Quick-save links, code snippets, or ideas. |
| [`obsidian-research`](skills/obsidian-research/) | "research and save", "look into this" | Research a topic via web/codebase, save structured findings. |
| [`obsidian-document`](skills/obsidian-document/) | "document this", "write docs on" | Write structured documentation about concepts, systems, or processes. |
| [`obsidian-blog`](skills/obsidian-blog/) | "write a blog on", "draft a blog post" | Draft blog posts with proper structure and frontmatter. |
| [`obsidian-guide`](skills/obsidian-guide/) | "write a guide", "create a tutorial" | Write step-by-step guides with verification and troubleshooting. |
| [`obsidian-meeting-notes`](skills/obsidian-meeting-notes/) | "save meeting notes", "capture this meeting" | Capture meetings with decisions, action items, and follow-ups. |
| [`obsidian-remember`](skills/obsidian-remember/) | "remember this", "note this down" | Lightest-weight capture for facts, decisions, and context. |

**Read skills** — retrieve and synthesize from the vault:

| Skill | Triggers | Description |
|-------|----------|-------------|
| [`obsidian-recall`](skills/obsidian-recall/) | "find my notes on X", "search obsidian" | Search and retrieve notes by topic, keyword, or tag. |
| [`obsidian-review`](skills/obsidian-review/) | "what have I been working on?", "recent sessions" | Browse recent vault activity grouped by date. |
| [`obsidian-lookup`](skills/obsidian-lookup/) | "do I have notes on X?", "did I already document X?" | Quick pre-check before starting work to avoid duplicates. |
| [`obsidian-digest`](skills/obsidian-digest/) | "summarize my notes on X", "brief me on X" | Synthesize multiple notes into a coherent summary. |

## Manual Installation

If you prefer not to use the CLI, clone and symlink:

```bash
# Clone the repo
git clone https://github.com/sarvesh-ghl/agent-skills.git

# Symlink a skill into your personal skills directory
# Cursor / Claude Code
mkdir -p ~/.cursor/skills
ln -s "$(pwd)/agent-skills/skills/pr-review" ~/.cursor/skills/pr-review

# Claude Code (standalone)
mkdir -p ~/.claude/skills
ln -s "$(pwd)/agent-skills/skills/pr-review" ~/.claude/skills/pr-review
```

Or copy the skill folder directly into your project:

```bash
cp -r agent-skills/skills/pr-review .cursor/skills/pr-review
```

## Repo Structure

```
agent-skills/
├── README.md
├── LICENSE                 # Apache 2.0
├── skills/
│   ├── pr-review/          # GitHub PR review
│   ├── qa-bug-fix-workflow/ # QA bug-fix from ClickUp
│   ├── obsidian-session-log/ # Coding session logs
│   ├── obsidian-bookmark/  # Quick-save links/snippets
│   ├── obsidian-research/  # Research and save findings
│   ├── obsidian-document/  # Structured documentation
│   ├── obsidian-blog/      # Blog post drafts
│   ├── obsidian-guide/     # Step-by-step guides
│   ├── obsidian-meeting-notes/ # Meeting notes
│   ├── obsidian-remember/  # Quick fact capture
│   ├── obsidian-recall/    # Search and retrieve notes
│   ├── obsidian-review/    # Browse recent activity
│   ├── obsidian-lookup/    # Pre-check for existing notes
│   └── obsidian-digest/    # Synthesize across notes
└── template/
    └── SKILL.md            # Starter template for new skills
```

## Creating a New Skill

Use the template to get started:

```bash
cp -r template skills/your-skill-name
```

Edit `skills/your-skill-name/SKILL.md` with your instructions. The frontmatter requires two fields:

```yaml
---
name: your-skill-name          # lowercase, hyphens, max 64 chars
description: What it does and when to use it.
license: Apache-2.0
metadata:
  author: your-github-username
  version: "1.0"
---
```

See the [Agent Skills Specification](https://agentskills.io/specification) for the full format reference.

## Contributing

Contributions are welcome! To add a skill:

1. Fork this repo
2. Create your skill in `skills/your-skill-name/SKILL.md`
3. Add an entry to the skills table in this README
4. Open a PR

Please ensure your skill:
- Follows the [Agent Skills spec](https://agentskills.io/specification)
- Has a specific, descriptive `description` field
- Keeps `SKILL.md` under 500 lines
- Includes concrete examples

## License

Apache 2.0 — see [LICENSE](LICENSE) for details.
