---
name: content
description: Content creation operations — topic suggestions from vault analysis, with room for future content workflows.
tags: [content, writing, topics, articles]
---

# content

Content creation and topic discovery from vault analysis.

## Modes

| Mode | Description |
|------|-------------|
| `suggest-topics` | Scan vault for topics with graph density, current energy, and synthesis potential |

## Invocation

```
/content suggest-topics
```

When invoked with no argument, display this help summary and ask which mode to run.

## Shared Dependencies

- `obsidian` CLI for vault search, backlinks, properties, and file analysis

## Mode: suggest-topics

**Resource:** `resources/suggest-topics.md`

Scan vault for topics with graph density, current energy, and synthesis potential to find what to write next.

**Quick reference:**
- Five-step workflow: Identify Well-Developed Topics → Identify "Alive" Topics → Score Synthesis Potential → Suggest Topics → Integration with topology
- Scoring: Development, Energy, Novelty, Clarity, Urgency (1-5 each, max 25)
- Minimum score of 15 to recommend, cap at 5 recommendations
- All claims must cite specific vault notes
