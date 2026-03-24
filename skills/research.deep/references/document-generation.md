# Document Generation Reference

Complete Phase 3 logic for research.deep skill: format selection, document generation, narrative review, and repository write.

---

## Phase 3 Overview

After synthesis is complete and user approves findings (Gate 1), Phase 3 handles:

1. Format selection (findings / six-pager / white-paper)
2. Draft generation from synthesis
3. Narrative review routing
4. User content approval (Gate 2)
5. Placement info collection (Gate 3)
6. Repository write (findings document + promoted notes + changelog)
7. Scratch space cleanup

---

## Step 8: Format Selection & Draft Generation

### Gate 1 Decision Point

After presenting synthesis, ask user:

```
What would you like to do with these findings?

1. Store — Generate a document and write to repository
2. Refine — Do another search/synthesis round
3. Discard — Delete scratch space and end session
```

**If user chooses "Refine"**:
- Ask: "What should I focus on in the next round?"
- Capture user direction
- Return to Phase 1 (search/gather) with refined scope
- Maintain existing scratch space, create new cycle subdirectory

**If user chooses "Discard"**:
- Confirm: "Delete scratch space and end session?"
- On confirmation: delete `{scratchSpaceRoot}/{session-dir}/`
- Report: "Session ended. Scratch space deleted."
- Stop

**If user chooses "Store"**:
- Proceed to format selection below

### Format Selection

Ask user:

```
Which format would you like?

1. findings — Overview, Findings, Sources, Open Questions (default)
2. six-pager — Amazon narrative: Intro, Problem, Tenets, Solution, Results, Appendices
3. white-paper — Abstract, Intro, Background, Analysis, Findings, Recommendations, References
```

Default to `findings` if user doesn't specify.

### Draft Generation

Generate draft in scratch space:

**File**: `{scratchSpaceRoot}/{session-dir}/draft.md`

**Content structure based on format**:
- Use synthesis from `synthesis.md`
- Use source notes from `sources/`
- Apply appropriate template (see Templates section below)

---

## Templates

### Format 1: Findings Document

```markdown
---
title: {Topic}
createdAt: {YYYY-MM-DD}
type: findings
tags: [research, {derived-tags}]
status: draft
---

# {Topic}

## Overview

{2-3 paragraph summary of what this research covers and why it matters}

## Key Findings

### {Theme 1 Title}

{2-4 paragraphs synthesizing Theme 1 from synthesis.md}

**Supporting Evidence**:
- {Evidence point from source notes}
- {Evidence point from source notes}

### {Theme 2 Title}

{2-4 paragraphs synthesizing Theme 2}

**Supporting Evidence**:
- {Evidence point}
- {Evidence point}

### {Theme 3 Title}

{Continue for all major themes...}

## Contradictions & Gaps

{If synthesis identified contradictions or gaps, document them here}

- **Contradiction**: {Description and which sources conflict}
- **Gap**: {What's missing and why it matters}

## Open Questions

{From synthesis open questions section}

1. {Question 1}
2. {Question 2}
3. {Question 3}

## Sources

{Alphabetical list of all sources from source notes}

1. **{Source Title}** — {URL or citation} — {1-sentence summary}
2. **{Source Title}** — {URL or citation} — {1-sentence summary}

---

**Research Date**: {YYYY-MM-DD}
**Sources Reviewed**: {N}
**Confidence**: {High | Medium | Low} — {1 sentence justification}
```

### Format 2: Six-Pager (Amazon Narrative)

```markdown
---
title: {Topic}
createdAt: {YYYY-MM-DD}
type: six-pager
tags: [research, narrative, {derived-tags}]
status: draft
---

# {Topic}

## Introduction

{What is this about? Why now? What's the goal?}

## Problem

{What problem does this solve or explore? What's broken or missing?}

## Tenets

{3-5 guiding principles or non-negotiables based on research findings}

1. {Tenet 1} — {Why it matters}
2. {Tenet 2} — {Why it matters}
3. {Tenet 3} — {Why it matters}

## Solution / Approach

{Based on synthesis: What works? What's recommended? What are the options?}

### Option 1: {Name}

{Description, pros, cons, evidence from sources}

### Option 2: {Name}

{Description, pros, cons, evidence from sources}

## Results & Evidence

{Synthesize supporting evidence from source notes}

- {Key finding with supporting data}
- {Key finding with supporting data}

## Open Questions

{From synthesis}

1. {Question}
2. {Question}

## Appendices

### Sources

{List all sources with URLs and summaries}

### Contradictions

{If applicable}

---

**Research Date**: {YYYY-MM-DD}
**Author**: {Your name or "AI Research Assistant"}
```

### Format 3: White Paper

```markdown
---
title: {Topic}
createdAt: {YYYY-MM-DD}
type: white-paper
tags: [research, white-paper, {derived-tags}]
status: draft
---

# {Topic}

## Abstract

{1 paragraph: What this paper covers, key findings, and conclusions}

## Introduction

{2-3 paragraphs: Context, scope, and purpose of this research}

## Background

{What you need to know to understand the topic. Include definitions, historical context, or prerequisite knowledge.}

## Analysis

### {Theme 1}

{In-depth analysis of Theme 1 with supporting evidence}

### {Theme 2}

{In-depth analysis of Theme 2 with supporting evidence}

### {Theme 3}

{Continue for all themes...}

## Findings

{Synthesize key takeaways from analysis section}

1. **{Finding 1}** — {Description with evidence}
2. **{Finding 2}** — {Description with evidence}
3. **{Finding 3}** — {Description with evidence}

## Recommendations

{Based on findings: What should be done? What are next steps?}

1. {Recommendation 1}
2. {Recommendation 2}
3. {Recommendation 3}

## Open Questions

{From synthesis}

1. {Question}
2. {Question}

## References

{Alphabetical bibliography of all sources}

1. {Author/Title/URL/Date}
2. {Author/Title/URL/Date}

---

**Research Date**: {YYYY-MM-DD}
**Status**: Draft
**Confidence Level**: {High | Medium | Low}
```

---

## Step 9: Narrative Review

After generating draft, route through narrative reviewer agent for writing quality check.

### Locate Narrative Reviewer

Check for narrative reviewer agent in this order:

1. `.kiro/agents/narrative-reviewer.md`
2. `.claude/agents/narrative-reviewer.md`

**If not found**:
- Log warning in `session.md`
- Skip review step
- Present unreviewed draft to user with warning

### Invoke Narrative Reviewer

**Pass to reviewer**:
- Draft content from `draft.md`
- Format type (findings / six-pager / white-paper)

**Reviewer returns**:
- Reviewed draft with corrections applied
- Review notes (what was changed and why)

**For complete review logic**, see: `references/narrative-review.md`

### Write Reviewed Draft

**File**: `{scratchSpaceRoot}/{session-dir}/reviewed-draft.md`

**Content**:
- Reviewed draft content (with corrections applied)
- Append review notes at bottom:

```markdown
---

## Review Notes

{Review notes from narrative-reviewer}
```

---

## Step 10: Content Approval (Gate 2)

Present reviewed draft to user:

```
Draft complete and reviewed. Please review:

{Path to reviewed-draft.md or display first 30 lines}

What would you like to do?

1. Approve — Write to repository
2. Edit — Make changes to draft (I'll wait)
3. Discard — Delete and end session
```

**If user chooses "Edit"**:
- User edits `reviewed-draft.md` directly
- When ready, user confirms: "Done editing"
- Proceed to Step 11

**If user chooses "Discard"**:
- Confirm: "Delete scratch space and end session?"
- On confirmation: delete `{scratchSpaceRoot}/{session-dir}/`
- Stop

**If user chooses "Approve"**:
- Proceed to Step 11

---

## Step 11: Placement Info & Repository Write (Gate 3)

### Collect Placement Info

**In standalone mode**, ask user:

```
Where should I write this document?

Please provide:
- Scope: S{ID}_{Title} (e.g., S01_Strategy)
- Parent note: P{ID}_{Title} or C{ID}_{Title} (optional)
- Suggested tags: {comma-separated list} (optional)
```

**In board mode**:
- Read placement info from board section metadata:
  - `Scope: S{scopeID}_{ScopeTitle}`
  - `Parent: P{parentID}_{ParentTitle}` (optional)
  - `Tags: tag1, tag2, tag3` (optional)

### Generate File Paths

**Main findings document**:
```
{scope-path}/C{scopeID}_{slug}.md
```

Where:
- `{scope-path}`: Resolve from scope ID (e.g., `S01_Strategy/`)
- `{slug}`: Topic slug (e.g., `bedrock-agents-vs-langgraph`)

**Promoted source notes** (if applicable):
```
{scope-path}/N{scopeID}_{source-slug}.md
```

### Write Findings Document

Read `reviewed-draft.md`, construct final document:

**Frontmatter**:
```yaml
---
title: {Topic}
createdAt: {YYYY-MM-DD}
updatedAt: {YYYY-MM-DD}
type: {findings | six-pager | white-paper}
scope: "[[S{ID}_{Title}]]"
parent: "[[P{ID}_{Title}]]"  # if provided
tags: [research, {user-provided-tags}, {derived-tags}]
status: complete
researchDate: {YYYY-MM-DD}
sourcesReviewed: {N}
confidence: {High | Medium | Low}
---
```

**Body**: Content from `reviewed-draft.md` (excluding review notes section)

**Wikilinks**:
- Add wikilinks to parent note if provided
- Add wikilinks to promoted source notes (if created)
- Add wikilinks to related content from synthesis

### Write Promoted Source Notes (Optional)

**Decision criteria**: Promote sources if:
- Source is particularly authoritative
- Source contains reusable reference material
- User requested note promotion

**For each promoted source**:

```markdown
---
title: {Source Title}
createdAt: {YYYY-MM-DD}
type: note
scope: "[[S{ID}_{Title}]]"
source: {URL}
tags: [source, {derived-tags}]
status: archived
relatedTo: ["[[C{ID}_{findings-title}]]"]
---

# {Source Title}

**Source**: {URL}
**Captured**: {YYYY-MM-DD}

## Summary

{1-2 paragraph summary from source note}

## Key Points

{Bullet list of key points from source note}

## Full Notes

{Complete content from source note in scratch space}

---

**Promoted from**: Research session `{session-dir}`
```

### Update Parent Note (If Provided)

If user provided parent note, append backlink:

Read parent note, append to `## Related` section (or create if missing):

```markdown
## Related

- [[C{ID}_{findings-title}]] — Research: {Topic} ({YYYY-MM-DD})
```

### Write Changelog Entry

Append to `filo-fax/changelog.md` (or create if missing):

```markdown
## {YYYY-MM-DD} — Research: {Topic}

- Created findings document: [[C{ID}_{slug}]]
- Sources reviewed: {N}
- Format: {findings | six-pager | white-paper}
- Scope: [[S{ID}_{Title}]]
- Confidence: {High | Medium | Low}
```

---

## Step 12: Cleanup

After successful repository write:

1. **Report success**:
   ```
   ✅ Research complete!

   Written to repository:
   - Findings: {path}/C{ID}_{slug}.md
   - Promoted notes: {N} sources
   - Changelog entry added

   Cleaning up scratch space...
   ```

2. **Delete scratch space**:
   ```bash
   rm -rf {scratchSpaceRoot}/{session-dir}/
   ```

3. **Confirm cleanup**:
   ```
   Session complete. Scratch space deleted.
   ```

---

## Board Mode Differences

In **board mode** (invoked by bb.exec):

### Summary Write (After Repository Write)

Write summary to board section under `### Cycle {N} Results`:

```markdown
### Cycle 1 Results
✅ Research complete — {N} sources reviewed

**Key Findings** (3-5 bullets):
- {Theme 1: 1-sentence summary}
- {Theme 2: 1-sentence summary}
- {Theme 3: 1-sentence summary}

**Open Questions**:
- {Question 1}
- {Question 2}

**Repository**: `{scope}/{path}/C{scopeID}_{title}.md`

**Next**: Review full document or refine research?
```

### Status Tag Update

Update board section status tag:
- `[ready]` → `[done:1]` (on success)
- `[ready]` → `[error:1]` (on failure)

### No Interactive Gates in Board Mode

Board mode skips interactive approval:
- Gate 1 (findings review): Auto-proceed
- Gate 2 (content review): Auto-proceed
- Gate 3 (placement info): Use board metadata

User reviews summary in board file, can trigger refinement by updating section and re-running.

---

## Error Handling

| Scenario | Behavior |
|----------|----------|
| Narrative reviewer unavailable | Skip review, warn user, proceed with unreviewed draft |
| Scope path not found | Report error, ask user to provide full path or correct scope ID |
| Parent note not found | Warn user, skip parent update, proceed with write |
| Repository write fails | Preserve scratch space for retry, report specific failures |
| Changelog write fails | Warn user, proceed (non-blocking) |
| Cleanup fails | Report error but consider session successful (files are in repo) |

---

## Notes

- This file contains **Phase 3 only** (Steps 8-12)
- For Phase 1-2 (search, synthesis), see `search-synthesis.md`
- For narrative review details, see `narrative-review.md`
- For board mode specifics, see `board-integration.md`
- Main skill file (`SKILL.md`) orchestrates all phases
