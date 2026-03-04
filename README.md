# Agent Skills

[![Skills](https://img.shields.io/badge/skills.sh-leaderboard-black)](https://skills.sh)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)

Reusable capabilities for AI coding agents. Each skill teaches an agent how to perform a specific task — reviewing PRs, generating docs, automating workflows — in a repeatable, high-quality way.

Compatible with **Cursor**, **Claude Code**, **Windsurf**, **Cline**, **Codex**, and [any agent that supports the Agent Skills spec](https://agentskills.io/specification).

## Quick Install

```bash
npx skills add sarvesh-ghl/agent-skills
```

To install a specific skill:

```bash
npx skills add sarvesh-ghl/agent-skills/pr-review
```

## Skills

| Skill | Description |
|-------|-------------|
| [`pr-review`](skills/pr-review/) | Review GitHub PRs with inline code comments via `gh` CLI. Fetches diff, analyzes changes, presents findings, iterates with you, and posts the review. |

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
│   └── pr-review/
│       └── SKILL.md        # Skill instructions + metadata
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
