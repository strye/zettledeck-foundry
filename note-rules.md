---
module: zettledeck-foundry
---

# Note Rules — Foundry

Rules for the note skill when operating within Foundry module workspace folders.

---

## Workspace Folders

| Role | Folder | Accept new notes? |
|------|--------|-------------------|
| `foundry` | `Foundry/` | Yes — ideation, content work, writing projects |
| `tesseract` | `Tesseract/` | No — managed exclusively by the `research` skill |

---

## Placement Rules

When a docType is identified, suggest the following folder by default:

| docType | Default role | Notes |
|---------|-------------|-------|
| `ideation` | `foundry` | Brainstorming and idea capture |
| `content` | `foundry` | Writing projects and produced artifacts; see delegation below |
| `research` | `foundry` | Research notes; see delegation below |

---

## Delegation Rules

When the user's intent matches the following patterns, offer to delegate to a specialist skill rather than creating a bare file. The user decides — the note skill does not invoke the specialist automatically.

### Research documents (`docType: research`)

- **Skill:** `/research {topic}`
- **Offer:** "The `research` skill has a full guided workflow — autonomous search, synthesis, and structured output with approval gates. Use it instead of a bare file?"
- **When:** User wants to research a topic. The `research` skill manages its own scratch space in Tesseract and produces structured output. If the user just wants a blank research note, proceed with bare creation in Foundry.

### Content writing (`docType: content`, or user intent is to write a post/essay/article)

- **Skill:** `/content write [topic]`
- **Offer:** "The `content` skill generates three writing versions (short post, personal essay, universal essay) from vault insights — use it instead of a bare file?"
- **When:** User wants to write a post, article, or essay. If the user just wants a blank content file, proceed with bare creation.

---

## Exclusions

| Role | Reason |
|------|--------|
| `tesseract` | Managed exclusively by the `research` skill as its scratch space. The note skill must never create files here directly. If a user requests Tesseract as a destination, redirect: "Tesseract is the research scratch space — it's managed by `/research`. Use Foundry for notes, or start a research session." |
