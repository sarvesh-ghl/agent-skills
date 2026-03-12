---
name: obsidian-blog
description: Draft a blog post and save it to the Obsidian vault. Use when the user says "write a blog on", "draft a blog post", "blog about", "write an article on", or wants to create long-form written content on a topic for publishing.
---

# Obsidian Blog

Draft a blog post on a given topic and save it to the Obsidian vault via the `user-obsidian` MCP server.

## Workflow

### Step 1: Understand the brief

Determine from conversation context:
- **Topic**: What the blog post is about
- **Audience**: Technical peers, general developers, non-technical readers
- **Tone**: Tutorial-style, opinion piece, deep dive, or storytelling
- **Key points**: What the user specifically wants covered
- **Length**: Short (~500 words), medium (~1000 words), or long (~2000+ words). Default: medium.

If the topic relates to the codebase or a recent coding session, draw on that context for concrete examples.

If the topic needs external research, use `WebSearch` and `WebFetch` to gather current information.

### Step 2: Build the draft

```markdown
# {Title}

> **Draft** — {YYYY-MM-DD}

---

## {Hook / Opening}

{Opening paragraph that establishes the problem or context. Make the reader care.}

## {Section 1 Title}

{Core content with examples, code blocks, or data}

## {Section 2 Title}

{Continue building the argument or tutorial}

## {Section 3+ as needed}

{Additional sections}

## Conclusion

{Wrap up with a clear takeaway or call to action}

---

*{Optional: author note, further reading links}*
```

### Step 3: Write to Obsidian

Save to `Projects/{Project Name}/` if project-specific, otherwise `Learning/`.

```json
{
  "path": "{vault_path}/Blog - {Title}.md",
  "content": "{blog content}",
  "frontmatter": {
    "tags": ["blog", "draft", "{topic-tags}"],
    "date": "{YYYY-MM-DD}",
    "type": "blog",
    "status": "draft",
    "audience": "{technical|general|non-technical}",
    "word-count": "{approximate}"
  },
  "mode": "overwrite"
}
```

### Step 4: Confirm

Report the note title, path, word count, and offer to refine specific sections.

## Rules

- **Lead with value.** Open with the problem or insight, not preamble.
- **Use concrete examples.** Code, screenshots, data -- not abstract claims.
- **One idea per section.** Each heading should earn its place.
- **Prefix filename with "Blog -"** to distinguish from other notes in the same folder.
- **Mark as draft** in frontmatter. The user will polish before publishing.
- Avoid filler phrases like "In this blog post, we will explore..." -- just start.
- If the topic is technical, include working code examples the reader can copy.
