---
name: research.deep
description: Research a topic across public and internal sources, gather structured notes in scratch space, synthesize findings, and produce schema-compliant documents for the repository. Supports both standalone and blackboard-driven modes.
---

# Research Deep — Autonomous Research & Synthesis

Research a topic across public and internal Amazon sources, gather structured notes in a scratch space, and produce schema-compliant findings documents for the Obsidian vault.

Operates autonomously within its scratch space — searching, reading, and synthesizing without per-step approval. User approval is required only at three gates: findings review, content review, and repository write.

**Supports two modes**:
- **Standalone mode**: Full autonomous workflow with 3 approval gates
- **Board mode**: Invoked by `bb.exec`, writes summary to board and full results to repository

## Invocation

**Standalone**:
```
/research.deep {topic}
/research.deep help
```

**Board mode** (invoked by bb.exec):
- Blackboard section marked `[ready]` with research topic
- bb.exec delegates to this skill
- Results written to board section + repository

## Help

If the user's input is the word "help" (case-insensitive), respond with:

```
🔍 Research Deep — Quick Reference

Researches a topic across public and internal Amazon sources, gathers
structured notes, and produces schema-compliant findings documents.

Usage:
  • {topic}           → research that topic (e.g. "Bedrock Agents vs LangGraph")
  • help              → this message

What it does:
  1. Searches public internet and internal Amazon sources (is.amazon.com)
  2. Reads and captures structured notes in a scratch space
  3. Self-reviews findings for completeness and gaps (autonomous)
  4. Synthesizes key themes across all sources
  5. Presents findings for your review (Gate 1)
  6. Generates a document in your chosen format (findings, six-pager, or white-paper)
  7. Routes draft through narrative-reviewer for writing quality (automatic)
  8. Presents reviewed draft for content approval (Gate 2)
  9. Collects placement info and writes to repository (Gate 3)

Approval gates:
  • Gate 1 — Review synthesized findings → store / refine / discard
  • Gate 2 — Review narrative-reviewed draft → approve / edit / discard
  • Gate 3 — Provide placement info → write to repository

Output formats:
  • findings     → Overview, Findings, Sources, Open Questions (default)
  • six-pager    → Amazon narrative: Intro, Problem, Tenets, Solution, Results, Appendices
  • white-paper  → Abstract, Intro, Background, Analysis, Findings, Recommendations, References

Scratch space:
  • Location read from config/config.json → researchScratchSpace (default: Tesseract/)
  • Each session gets its own directory: {YYYYMMDD}-{slug}/
  • Scratch space is deleted after storage or discard

Board mode:
  • When invoked by bb.exec from a blackboard section
  • Full results written to repository
  • Summary + questions written back to board section
  • Board section status updated through lifecycle

Notes:
  • The assistant operates autonomously between approval gates
  • All intermediate file operations stay in the scratch space
  • No repository files are written without your explicit approval
```

Then stop — do not proceed to the steps below.

## Requirements

**MCP Servers**:
- `remote_web_search` (or similar) - Public web search
- `mcp_amzn_mcp_search_internal_websites` - Internal Amazon search
- `mcp_amzn_mcp_read_internal_website` - Internal page reads
- `webFetch` - Public page reads

**Tools needed**: Read, Write, Glob

**Referenced files**:
- `config/config.json` - Scratch space path configuration
- `.kiro/agents/narrative-reviewer.md` or `.claude/agents/narrative-reviewer.md` - Document review
- Vault steering files - For markdown conventions

**Mode-specific**:
- **Board mode**: Active blackboard file with research section

## Mode Detection

**At skill invocation**, determine the operating mode:

### Standalone Mode
**Triggers**:
- User invokes skill directly: `/research.deep {topic}`
- No board context provided
- User explicitly requests standalone operation

**Behavior**:
- Full 3-gate approval workflow
- User provides placement info at Gate 3
- Results written only to repository
- Scratch space cleaned up after write

### Board Mode
**Triggers**:
- Invoked by `bb.exec` with board context
- Board section contains:
  - Research topic
  - Optional scope/placement hints
  - Status tag `[ready]` or `[ready:N]`

**Behavior**:
- Autonomous phases 1-2 (search, synthesis)
- Brief findings summary written to board section
- User reviews summary in board file (lightweight Gate 1/2)
- Full document written to repository (Gate 3 with board-provided placement)
- Board section status updated to `[done]` or `[error]`

**Board mode differences**:
- No interactive gates during search/synthesis (fully autonomous)
- Placement info pre-configured in board section or auto-derived
- Summary to board, full results to repository
- Status tags managed by board lifecycle

## Configuration

At the start of every research session, resolve the scratch space path before doing anything else.

**See `references/search-synthesis.md` for detailed configuration steps.**

**Summary**:
1. Read `config/config.json` and parse `researchScratchSpace` key
2. Default to `Tesseract/` if config missing or key not present
3. Validate path is NOT inside a scope folder (reject `S\d+_*` pattern)
4. Create scratch space directory if it doesn't exist

**Resolved path**: `{scratchSpaceRoot}/{YYYYMMDD}-{slug}/`

---

## Workflow Overview

### Phase 1: Search, Gather & Self-Review
1. Parse topic and create session directory
2. Search public + internal sources (parallel)
3. Read pages and create source notes
4. **Self-review loop** (autonomous): Fill gaps, resolve contradictions, clarify terms
5. Present sources to user (or skip if board mode)

**See `references/search-synthesis.md` for complete Phase 1-2 logic.**

### Phase 2: Synthesize
1. Build running synthesis from source notes
2. Identify themes, patterns, contradictions, open questions
3. Present synthesis for Gate 1 review
4. User chooses: store / refine / discard

**Board mode**: Write summary to board section, proceed to Phase 3 automatically

### Phase 3: Store or Discard
1. User selects output format (findings / six-pager / white-paper)
2. Generate draft from synthesis
3. Route through narrative-reviewer agent (automatic)
4. Present reviewed draft for Gate 2 approval
5. Collect placement info (Gate 3)
6. Write findings document + promoted notes + changelog to repository
7. Clean up scratch space

**Board mode**: Use board section metadata for placement, write full results to repository + summary to board

**See `references/document-generation.md` for templates and Phase 3 logic.**
**See `references/narrative-review.md` for review workflow.**
**See `references/board-integration.md` for board mode specifics.**

---

## Autonomous Operation Rules

### Scratch Space Autonomy
- Full read/write access to `{scratchSpaceRoot}/{session-dir}/`
- No user confirmation needed for scratch operations
- All intermediate work (source notes, synthesis, drafts) stays in scratch

### Repository Protection
- SHALL NOT write outside scratch space without explicit user approval
- User approval required at exactly three gates:
  1. **Gate 1** — Review findings, choose store/refine/discard
  2. **Gate 2** — Review narrative-reviewed draft, approve/edit/discard
  3. **Gate 3** — Provide placement info, confirm repository write

**Board mode exception**: Gates 1-2 are lightweight (summary review in board), Gate 3 uses board metadata

### Parallel Execution
- Issue independent tool calls in parallel (searches, page reads)
- Don't serialize independent operations
- Reduces session time

### Error Resilience
- Log errors in `session.md` under `## Errors`
- Continue with remaining tasks (partial results are valuable)
- Only stop if all search tools fail simultaneously

### Self-Review Loop
- Autonomous gap-filling (no user approval during loop)
- Maximum 3 iterations
- Targets: coverage gaps, undefined terms, contradictions, unanswered questions

---

## Error Handling

| Scenario | Behavior |
|----------|----------|
| `remote_web_search` fails | Log error, continue with internal search |
| Internal search fails | Log error, continue with public results |
| Page read fails | Skip source, mark failed, continue others |
| Both searches fail | Report failure, suggest retry or manual URLs, stop |
| No results from searches | Report, suggest refined keywords, stop |
| Scratch space conflicts with scope folder | Reject path, report error, stop session |
| Config missing/unreadable | Default to `Tesseract/`, warn user |
| Session directory exists | Append numeric suffix `-2`, `-3`, etc. |
| Narrative reviewer unavailable | Skip review, present unreviewed draft with warning |
| Repository write fails | Preserve scratch space for retry, report failures |
| Board section missing required fields | Ask user to update board section, stop |

---

## Board Mode Integration

**For complete board mode workflow, see `references/board-integration.md`.**

### Board Section Format
```markdown
## 🔍 Research: {Topic} [ready]
<!-- agent-id: research.deep | cycle: 1 | updated: 2026-03-15T10:00:00 -->

### Recommendations
- Topic: "{research topic}"
- Output format: findings | six-pager | white-paper
- Scope: S{scopeID}_{ScopeTitle}
- Parent: P{parentID}_{ParentTitle} (optional)
- Tags: tag1, tag2, tag3 (optional)

### Response
{user can provide additional context or refinements}

### Cycle {N} Results
{research.deep writes summary here after execution}
```

### Board Mode Execution Flow
1. bb.exec reads section marked `[ready]`
2. bb.exec invokes `research.deep` with board context
3. research.deep:
   - Runs Phase 1-2 autonomously (search, synthesis)
   - Generates full document
   - Routes through narrative review
   - Writes full document to repository using board metadata
   - Writes summary to board section under `### Cycle {N} Results`
   - Updates board section status to `[done:N]` or `[error:N]`
4. User reviews summary in board
5. If needs refinement, user updates section and re-runs

### Summary Format (Written to Board)
```markdown
### Cycle 1 Results
✅ Research complete — {N} sources reviewed

**Key Findings** (3-5 bullets):
- {Theme 1 summary}
- {Theme 2 summary}
- {Theme 3 summary}

**Open Questions**:
- {Question 1}
- {Question 2}

**Repository**: `{scope}/{path}/C{scopeID}_{title}.md`

**Next**: Review full document or refine research?
```

---

## Implementation Notes

This skill delegates detailed logic to reference files using progressive disclosure:

- **Phase 1-2**: See `references/search-synthesis.md` for search, read, self-review, synthesis
- **Phase 3**: See `references/document-generation.md` for templates, narrative review, repository write
- **Board mode**: See `references/board-integration.md` for board lifecycle, section parsing, result writing
- **Review**: See `references/narrative-review.md` for tier-based review model

**Main skill handles**: Mode detection, configuration, high-level workflow orchestration, error handling

**References handle**: Detailed step-by-step logic, format templates, edge cases

This structure keeps the main skill readable while preserving all functionality from the original 1,563-line prompt.

---

## Future Capabilities

**Planned expansions** for `research.*` domain:
- `research.quick` - Single-source summary (no synthesis)
- `research.compare` - A/B comparison focused (2 specific items)
- `research.monitor` - Ongoing topic tracking (scheduled searches)
- `research.validate` - Fact-checking existing content

**Board integration** ready for all future research skills - same section format, different execution logic.
