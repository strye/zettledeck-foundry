---
module: foundry
order: 40
description: Configure research output path and optional blackboard integration
---

## Research Output Path

**What:** The directory where research documents and synthesized notes are saved.
**File:** `skills/research/SKILL.md`
**Marker:** `{research-path}`
**Default:** `Tesseract` (dedicated research workspace)

### Questions to ask the user

1. Where should research documents be saved?
   - `Tesseract` (default — dedicated research workspace)
   - A scope-specific path (e.g., `S03_Work/research`)
   - Enter a custom path

### How to apply

Replace `{research-path}` in `skills/research/SKILL.md`. The research skill saves synthesized outputs as `{research-path}/{topic-slug}.md`.

---

## Blackboard Integration (Optional)

**What:** Whether to enable rostrum-blackboard integration for persistent research context across sessions.
**File:** `skills/research/references/board-integration.md`
**Marker:** `{blackboard-enabled}`
**Default:** false (disabled)

### Questions to ask the user

1. Do you have rostrum-blackboard installed?
   - No — skip this step (default)
   - Yes — configure integration

### How to apply

If enabled: Set `{blackboard-enabled}` to `true` in the board-integration reference file. The research skill will check for an active blackboard at the start of each session and write research notes to it.

If disabled: Note in the init summary that blackboard integration is available if they install `rostrum-blackboard` later.
