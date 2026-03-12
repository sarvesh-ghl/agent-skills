---
name: obsidian-meeting-notes
description: Capture and save meeting notes to the Obsidian vault. Use when the user says "save meeting notes", "store meeting notes", "capture this meeting", "log meeting", "meeting summary", or shares meeting content to be persisted.
---

# Obsidian Meeting Notes

Capture meeting notes in a structured format and save to the Obsidian vault via the `user-obsidian` MCP server.

## Workflow

### Step 1: Extract meeting info

From conversation context, extract:
- **Title/Topic**: What the meeting was about
- **Date**: When it happened (default: today)
- **Attendees**: Who was there (if mentioned)
- **Type**: Standup, planning, retrospective, 1:1, design review, incident review, ad-hoc
- **Key discussion points**: What was discussed
- **Decisions made**: What was decided and by whom
- **Action items**: Who needs to do what by when
- **Follow-ups**: Things to revisit later

### Step 2: Build the note

```markdown
# {Meeting Title}

> **Date:** {YYYY-MM-DD}
> **Type:** {meeting type}
> **Attendees:** {comma-separated names}

---

## Summary

{2-3 sentence overview of what was discussed and the outcome}

## Discussion

### {Topic 1}

{Key points discussed}

### {Topic 2}

{Key points discussed}

## Decisions

| Decision | Rationale | Owner |
|----------|-----------|-------|
| {What was decided} | {Why} | {Who} |

## Action Items

- [ ] {Task description} — @{owner} (due: {date if known})
- [ ] {Task description} — @{owner}

## Follow-ups

- {Topic to revisit in next meeting}
- {Open question that needs more info}
```

Omit empty sections.

### Step 3: Write to Obsidian

```json
{
  "path": "Meeting Notes/{YYYY-MM-DD} - {Meeting Title}.md",
  "content": "{note content}",
  "frontmatter": {
    "tags": ["meeting", "{meeting-type}", "{project-tag-if-relevant}"],
    "date": "{YYYY-MM-DD}",
    "type": "meeting-notes",
    "meeting-type": "{standup|planning|retro|1on1|design-review|incident-review|ad-hoc}",
    "attendees": ["{name1}", "{name2}"]
  },
  "mode": "overwrite"
}
```

**Filename format**: `{YYYY-MM-DD} - {Meeting Title}.md` for chronological sorting.

### Step 4: Confirm

Report the note title, path, number of action items, and number of decisions captured.

## Rules

- **Decisions are the most important section.** Capture who decided what and why.
- **Action items need owners.** A task without an owner won't get done.
- **Date-prefix the filename** for chronological sorting in the vault.
- **Be concise.** Meeting notes are for reference, not transcripts. Distill to what matters.
- Link related project notes with `[[Note Name]]` wikilinks.
- If the meeting is about a specific project, add the project tag (e.g., `project/workflows-backend`).
