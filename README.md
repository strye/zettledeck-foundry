# zettledeck-foundry

Content creation and deep research module for ZettleDeck. Research workflows, professional communications, content suggestions, writing from vault insights, and document versioning.

---

## What's Included

### Skills

| Skill | Modes | Description |
|-------|-------|-------------|
| `research.deep` | (multi-phase) | Research topics across public and internal sources, synthesize into schema-compliant documents |
| `comms.compose` | (iterative) | Draft professional correspondence with file-based workspace and conversation tracking |
| `learning` | `write`, `compile` | Generate writing pieces and compile weekly learnings from vault insights |
| `content` | `suggest-topics` | Find what to write next by analyzing vault density, energy, and synthesis potential |
| `content-versioning` | (reference) | Background knowledge for content editing — source protection, working copies, changelogs |

---

## Installation

### Via zd (recommended)

```bash
zd install foundry
```

Then run `/zettledeck.init foundry` to configure paths and optional integrations.

### Manual

1. Copy `skills/` into your `.shared/skills/` directory
2. Configure paths marked `{placeholder}` in SKILL.md files

---

## Configuration

After installation, run:

```
/zettledeck.init foundry
```

This walks you through:
- **Research output path** — where research documents are saved
- **Comms workspace path** — where draft correspondence files are stored
- **Optional:** blackboard integration for multi-session research continuity

---

## Quick Start

```
/research.deep                    — Start a deep research session
/research.deep topic              — Research a specific topic

/comms.compose                    — Start a new draft
/comms.compose path/to/draft.md   — Continue working on an existing draft

/learning write                   — Generate writing from today's insights
/learning write topic             — Generate writing on a specific topic
/learning compile                 — Compile weekly learnings for team

/content suggest-topics           — Find what to write next from vault analysis
```

---

## Optional Integration

**rostrum-blackboard:** The `research.deep` skill integrates with a shared blackboard for multi-session research continuity. When a blackboard is active, research notes and source tracking persist across sessions. Install `rostrum-blackboard` to enable.

**zettledeck-nexus:** The `learning write` and `learning compile` modes pair well with `ideas.generate` and `vault.build-context` from nexus for a complete content creation pipeline.

---

## Content Versioning

All content editing in Foundry follows a protect-the-source pattern:
- Source documents are never modified during iteration
- Working copies live in `_versions/` subfolders
- Changelogs track every version with context
- Explicit promotion step to publish revisions

The `content-versioning` skill auto-loads as background knowledge whenever you edit, revise, or iterate on content files.

---

## Part of ZettleDeck

This module is part of the [ZettleDeck](https://github.com/your-org/zettledeck-core) ecosystem. Core functionality requires `zettledeck-core`.

Other modules: [zettledeck-almanac](../zettledeck-almanac/) · [zettledeck-nexus](../zettledeck-nexus/)
