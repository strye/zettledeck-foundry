---
module: foundry
order: 40
description: Configure research output path, comms workspace path, and optional blackboard integration
---

## Research Output Path

**What:** The directory where research documents and synthesized notes are saved.
**File:** `skills/research.deep/SKILL.md`
**Marker:** `{research-path}`
**Default:** `research` (a top-level vault folder)

### Questions to ask the user

1. Where should research documents be saved?
   - `research` (default — top-level vault folder)
   - A scope-specific path (e.g., `S03_Work/research`)
   - Enter a custom path

### How to apply

Replace `{research-path}` in `skills/research.deep/SKILL.md`. The research skill saves synthesized outputs as `{research-path}/{topic-slug}.md`.

---

## Comms Workspace Path

**What:** The directory where draft correspondence files are created and tracked. The comms.compose skill uses file-based workspaces for iterative drafting.
**File:** `skills/comms.compose/SKILL.md`
**Marker:** `{comms-path}`
**Default:** `drafts`

### Questions to ask the user

1. Where should draft correspondence files be stored?
   - `drafts` (default — top-level vault folder)
   - `filo-fax/drafts`
   - Enter a custom path

### How to apply

Replace `{comms-path}` in `skills/comms.compose/SKILL.md`. Each draft creates a file at `{comms-path}/{slug}.md` with frontmatter tracking status, channel, and conversation history.

---

## Blackboard Integration (Optional)

**What:** Whether to enable rostrum-blackboard integration for persistent research context across sessions.
**File:** `skills/research.deep/references/board-integration.md`
**Marker:** `{blackboard-enabled}`
**Default:** false (disabled)

### Questions to ask the user

1. Do you have rostrum-blackboard installed?
   - No — skip this step (default)
   - Yes — configure integration

### How to apply

If enabled: Set `{blackboard-enabled}` to `true` in the board-integration reference file. The research skill will check for an active blackboard at the start of each session and write research notes to it.

If disabled: Note in the init summary that blackboard integration is available if they install `rostrum-blackboard` later.

---

## Weekly Learnings Diary Path

**What:** The path to daily diary files, used by `learning compile` to read the week's notes.
**File:** `skills/learning/resources/compile.md`
**Marker:** `{diary-path}`
**Default:** `filo-fax/diary`

### Questions to ask the user

1. Where are your daily diary files? (Same path as almanac, if installed)
   - `filo-fax/diary` (default)
   - Enter a custom path

### How to apply

Replace `{diary-path}` in `skills/learning/resources/compile.md`. The compile mode reads `{diary-path}/YYYY/MMM/` to find the week's daily files.
