---
module: foundry
order: 40
description: Configure research output path and optional blackboard integration
---

## Configuration Convention

All foundry configuration is stored in `.zettledeck/zettledeck-foundry/config.json`. This file is the single source of truth for research skill runtime configuration.

### Config file format

**File:** `.zettledeck/zettledeck-foundry/config.json`

```json
{
  "researchScratchSpace": "Tesseract/"
}
```

---

## Research Output Path

**What:** The directory where research scratch space and synthesized documents are saved.
**Config key:** `researchScratchSpace`
**Default:** `Tesseract/` (dedicated research workspace)

### Questions to ask the user

1. Where should research documents be saved?
   - `Tesseract` (default — dedicated research workspace)
   - A scope-specific path (e.g., `S03_Work/research`)
   - Enter a custom path

### How to apply

1. Write the user's choice to `researchScratchSpace` in `.zettledeck/zettledeck-foundry/config.json`
2. The research skill reads this config at runtime — no placeholder replacement needed in skill files
3. Validate the path is NOT inside a scope folder (reject `S\d+_*` pattern)

---

## Blackboard Integration (Optional)

**What:** Whether rostrum-blackboard is available for board-mode research.
**No config key** — the research skill auto-detects board mode when invoked by bb.exec.

### Questions to ask the user

1. Do you have rostrum-blackboard installed?
   - Check `.zettledeck/rostrum-blackboard/` for existence
   - If present: note that board-mode research is available
   - If absent: note that it can be added later via `zd install`

### How to apply

No config changes needed. Board mode is detected at runtime by bb.exec passing board context to the research skill. Just inform the user of the integration status.
