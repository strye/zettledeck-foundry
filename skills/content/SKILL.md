---
name: content
description: Content creation — topic suggestions from vault analysis, generate writing pieces from vault insights, and content versioning workflow for editing and iterating on content files.
---

# content

Content creation, topic discovery, writing from vault insights, and content versioning.

## Modes

| Mode | Description |
|------|-------------|
| `suggest-topics` | Scan vault for topics with graph density, current energy, and synthesis potential |
| `write` | Generate three writing versions (short post, personal essay, universal essay) from a vault insight or topic |

## References

| Reference | Loaded when... |
|-----------|----------------|
| `references/content-versioning.md` | Editing, revising, iterating on, or reviewing content files |

## Invocation

```
/content suggest-topics
/content write [topic]
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

## Mode: write

**Resource:** `resources/write.md`

Generate three writing versions from a vault insight or topic.

**Quick reference:**
- Six-step workflow: Identify Topic → Mine Material → External Research → Non-Obvious Angle → Generate Three Versions → Solicit Feedback
- Outputs: Short post (1-3 paragraphs), Personal essay (300-600 words), Universal essay (800-1500 words)
- Accept explicit topic or auto-detect from today's daily note
