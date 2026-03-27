# zettledeck-foundry

Content creation and deep research module for ZettleDeck. Research workflows, professional communications, content suggestions, writing from vault insights, and document versioning.

---

## What's Included

### Skills

| Skill | Modes | Description |
|-------|-------|-------------|
| `research` | (multi-phase) | Research topics across public and internal sources, synthesize into schema-compliant documents |
| `comms` | (iterative) | Draft professional correspondence with file-based workspace and conversation tracking |
| `content` | `suggest-topics`, `write` | Content creation — topic suggestions from vault analysis and writing pieces from vault insights. Includes content versioning reference. |

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
/research                    — Start a deep research session
/research topic              — Research a specific topic

/comms                    — Start a new draft
/comms path/to/draft.md   — Continue working on an existing draft

/content suggest-topics           — Find what to write next from vault analysis
/content write                    — Generate writing from today's insights
/content write topic              — Generate writing on a specific topic
```

---

## Optional Integration

**rostrum-blackboard:** The `research` skill integrates with a shared blackboard for multi-session research continuity. When a blackboard is active, research notes and source tracking persist across sessions. Install `rostrum-blackboard` to enable.

**zettledeck-nexus:** The `content write` mode pairs well with `ideas.generate` and `vault.build-context` from nexus for a complete content creation pipeline.

---

## Content Versioning

All content editing in Foundry follows a protect-the-source pattern:
- Source documents are never modified during iteration
- Working copies live in `_versions/` subfolders
- Changelogs track every version with context
- Explicit promotion step to publish revisions

The content versioning reference (`content/references/content-versioning.md`) is loaded when editing, revising, or iterating on content files.

---

## Part of ZettleDeck

This module is part of the [ZettleDeck](https://github.com/your-org/zettledeck-core) ecosystem. Core functionality requires `zettledeck-core`.

Other modules: [zettledeck-praxis](../zettledeck-praxis/) · [zettledeck-nexus](../zettledeck-nexus/)
