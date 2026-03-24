---
name: learning
description: Generate writing pieces from vault insights and compile weekly learnings for team communications. Turns accumulated knowledge into shareable content.
version: 1.0.0
tags: [learning, writing, essays, weekly, synthesis, content]
---

# learning

Learning capture and weekly learnings compilation for content creation.

## Modes

| Mode | Command | Description |
|------|---------|-------------|
| `write` | `/learning write [topic]` | Generate three writing versions (short post, personal essay, universal essay) from vault insights |
| `compile` | `/learning compile` | Synthesize a week's work into structured team communications |

## Invocation

```
/learning write [topic]
/learning compile
```

When invoked with no argument, display this help summary and ask which mode to run.

## Shared Dependencies

- `obsidian` CLI for vault search, backlinks, properties, and daily note access

## Mode: write

**Resource:** `resources/write.md`

Generate three writing versions from a vault insight or topic.

**Quick reference:**
- Six-step workflow: Identify Topic → Mine Material → External Research → Non-Obvious Angle → Generate Three Versions → Solicit Feedback
- Outputs: Short post (1-3 paragraphs), Personal essay (300-600 words), Universal essay (800-1500 words)
- Accept explicit topic or auto-detect from today's daily note

## Mode: compile

**Resource:** `resources/compile.md`

Synthesize a week's work into structured team communications.

**Quick reference:**
- Seven-step process: Historical Context → Daily Note Analysis → Strategic Context → Idea Evolution → External Signals → Calendar/Messages → Terminal Output
- Output: WEEKLY LEARNINGS PREP with candidate topics ranked by depth
- Key principle: best weekly learnings come from noticing patterns the writer was too close to see

> **Note:** The `identify-skills` mode for finding high-leverage learning opportunities is available in the `zettledeck-nexus` module.
