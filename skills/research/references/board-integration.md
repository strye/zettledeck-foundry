# Board Integration Reference

Complete specification for research integration with bb.exec (Blackboard) workflow.

---

## Overview

**research** supports two operating modes:

1. **Standalone mode** — User invokes directly, three interactive approval gates
2. **Board mode** — Invoked by bb.exec, autonomous execution with summary feedback

This file specifies board mode behavior: section format, execution flow, result writing, and status lifecycle.

---

## Mode Detection

At skill invocation, determine operating mode:

### Standalone Mode Triggers
- User invokes skill directly: `/research {topic}`
- No board context provided in invocation
- User explicitly requests standalone operation

### Board Mode Triggers
- Invoked by `bb.exec` with board context
- Board section contains:
  - Research topic
  - Optional scope/placement hints
  - Status tag `[ready]` or `[ready:N]`

**Detection mechanism**: bb.exec passes board context as parameter:

```
Board context:
- Section: "🔍 Research: {Topic}"
- Status: [ready:1]
- Metadata: {scope, parent, tags, output format}
```

If board context is present → **Board mode**
If board context is absent → **Standalone mode**

---

## Board Section Format

Blackboard sections for research follow this structure:

```markdown
## 🔍 Research: {Topic} [ready]
<!-- agent-id: research | cycle: 1 | updated: 2026-03-15T10:00:00 -->

### Recommendations

- Topic: "{research topic or question}"
- Output format: findings | six-pager | white-paper
- Scope: S{scopeID}_{ScopeTitle}
- Parent: P{parentID}_{ParentTitle} (optional)
- Tags: tag1, tag2, tag3 (optional)

### Response

{user can provide additional context, refinement direction, or constraints}

### Cycle 1 Results

{research writes summary here after execution}
```

### Field Descriptions

**Section heading**:
- Emoji: 🔍 (indicates research task)
- Topic: Short title (e.g., "Bedrock Agents vs LangGraph")
- Status tag: `[ready]` or `[ready:N]` (N = cycle number)

**HTML comment** (metadata):
- `agent-id`: Always `research` for research tasks
- `cycle`: Current iteration number (1, 2, 3...)
- `updated`: ISO timestamp of last update

**Recommendations subsection**:
- `Topic`: Full research question or topic description
- `Output format`: One of: findings, six-pager, white-paper (default: findings)
- `Scope`: Target scope for repository write (required)
- `Parent`: Parent note for backlink (optional)
- `Tags`: Additional tags for frontmatter (optional)

**Response subsection**:
- User-provided context or refinement direction
- Optional, can be empty
- Used to guide search strategy or synthesis focus

**Cycle N Results subsection**:
- Created/updated by research after execution
- Contains summary of findings, open questions, repository path
- Numbered by cycle (Cycle 1, Cycle 2, etc.)

---

## Board Mode Execution Flow

### Step 0: bb.exec Invocation

bb.exec (Blackboard executor) reads board file, identifies section marked `[ready]`:

```markdown
## 🔍 Research: Bedrock Agents vs LangGraph [ready:1]
```

bb.exec invokes research with board context:

```
Invoke: research
Board context:
  - Section: "Research: Bedrock Agents vs LangGraph"
  - Topic: "Compare Amazon Bedrock Agents and LangGraph for agentic workflows"
  - Output format: findings
  - Scope: S05_Technology
  - Parent: P05_AI_Systems
  - Tags: [agents, bedrock, langgraph, comparison]
  - Cycle: 1
  - Status: [ready:1]
```

### Step 1: Update Status to Executing

research updates board section status tag:

```markdown
## 🔍 Research: Bedrock Agents vs LangGraph [executing:1]
<!-- agent-id: research | cycle: 1 | updated: 2026-03-15T10:05:00 -->
```

**Purpose**: Signals to user/system that work is in progress.

### Step 2: Run Phase 1-2 Autonomously

Execute search, gather, self-review, synthesis **without user interaction**:

- Phase 1: Search public + internal sources (parallel)
- Phase 1: Read pages, create source notes in scratch space
- Phase 1: Self-review loop (autonomous, max 3 iterations)
- Phase 2: Build synthesis from source notes
- Phase 2: Identify themes, contradictions, open questions

**No approval gates during Phases 1-2 in board mode.**

All work stays in scratch space: `{scratchSpaceRoot}/{session-dir}/`

### Step 3: Generate Document (Phase 3)

Autonomously generate document:

- Use output format from board metadata (findings / six-pager / white-paper)
- Generate draft from synthesis
- Route through narrative reviewer (if available)
- No Gate 2 approval — proceed automatically

### Step 4: Write to Repository

Use placement info from board metadata:

- Scope: From board section `Scope:` field
- Parent: From board section `Parent:` field (if provided)
- Tags: From board section `Tags:` field + derived tags

Write files:
- Main findings document: `{scope-path}/C{scopeID}_{slug}.md`
- Promoted source notes (if applicable): `{scope-path}/N{scopeID}_{source-slug}.md`
- Changelog entry: `Praxis/changelog.md`
- Update parent note backlinks (if parent provided)

### Step 5: Write Summary to Board Section

Under `### Cycle {N} Results`, write summary:

```markdown
### Cycle 1 Results
✅ Research complete — 12 sources reviewed

**Key Findings** (3-5 bullets):
- Bedrock Agents excel at integration with AWS services and enterprise tooling
- LangGraph offers more flexibility for complex multi-agent workflows
- Both support similar agentic patterns (ReAct, planning, memory)
- Bedrock has tighter guardrails and compliance features
- LangGraph has stronger open-source community and extensibility

**Open Questions**:
- Cost comparison at scale (100k+ agent invocations/month)?
- Migration path from LangGraph to Bedrock Agents?

**Repository**: `S05_Technology/C05_bedrock-agents-vs-langgraph.md`

**Next**: Review full document or refine research?
```

### Step 6: Update Status Tag

Update board section status:

**On success**:
```markdown
## 🔍 Research: Bedrock Agents vs LangGraph [done:1]
<!-- agent-id: research | cycle: 1 | updated: 2026-03-15T10:42:00 -->
```

**On error**:
```markdown
## 🔍 Research: Bedrock Agents vs LangGraph [error:1]
<!-- agent-id: research | cycle: 1 | updated: 2026-03-15T10:15:00 -->
```

### Step 7: Cleanup Scratch Space

Delete scratch space directory: `{scratchSpaceRoot}/{session-dir}/`

**Exception**: On error, preserve scratch space for debugging.

### Step 8: Return to bb.exec

research reports completion to bb.exec:

```
✅ Research complete
- Status: done:1
- Repository: S05_Technology/C05_bedrock-agents-vs-langgraph.md
- Summary written to board section
```

bb.exec logs result and proceeds to next board section (if any).

---

## Summary Format Specification

The summary written to `### Cycle {N} Results` follows this template:

```markdown
### Cycle {N} Results
✅ Research complete — {source-count} sources reviewed

**Key Findings** (3-5 bullets):
- {Theme 1: 1-sentence summary from synthesis}
- {Theme 2: 1-sentence summary from synthesis}
- {Theme 3: 1-sentence summary from synthesis}
- {Theme 4: 1-sentence summary from synthesis} (if applicable)
- {Theme 5: 1-sentence summary from synthesis} (if applicable)

**Contradictions** (if any):
- {Contradiction summary}

**Open Questions**:
- {Question 1 from synthesis}
- {Question 2 from synthesis}
- {Question 3 from synthesis} (if applicable)

**Repository**: `{scope-path}/C{scopeID}_{slug}.md`

**Confidence**: {High | Medium | Low} — {1-sentence justification}

**Next**: Review full document or refine research?
```

### Field Guidelines

**Key Findings**:
- 3-5 bullets minimum
- Each bullet is 1-2 sentences max
- Captures essence of each major theme from synthesis
- Actionable and specific (not vague)

**Contradictions** (optional):
- Only include if synthesis identified meaningful contradictions
- 1-2 bullets max
- State the conflict and which sources disagree

**Open Questions**:
- 2-5 bullets
- Questions from synthesis that remain unanswered
- Specific enough to guide follow-up research

**Repository path**:
- Full path relative to vault root
- Uses wikilink format: `[[C{ID}_{slug}]]` or plain path

**Confidence**:
- High: 10+ authoritative sources, consistent findings, recent data
- Medium: 5-9 sources, some gaps or contradictions, mixed recency
- Low: <5 sources, significant gaps, outdated data, or high uncertainty

**Next**:
- Always end with: "Review full document or refine research?"
- Prompts user to decide on iteration or acceptance

---

## Refinement Workflow

If user wants to refine research after reviewing summary:

### User Action

User edits board section:

1. Updates `### Response` with refinement direction:
   ```markdown
   ### Response
   Please focus more on cost comparison and migration paths.
   Deprioritize community/extensibility aspects.
   ```

2. Updates status tag from `[done:1]` to `[ready:2]`:
   ```markdown
   ## 🔍 Research: Bedrock Agents vs LangGraph [ready:2]
   ```

3. Re-runs bb.exec

### research Action

On second invocation:

1. Detects cycle 2 from status tag `[ready:2]`
2. Reads user direction from `### Response`
3. Creates new scratch space: `{scratchSpaceRoot}/{YYYYMMDD}-{slug}-cycle2/`
4. Runs Phase 1-2 with refined focus
5. Merges findings with previous cycle (if applicable)
6. Writes updated document to repository (overwrites previous)
7. Writes new summary under `### Cycle 2 Results`
8. Updates status to `[done:2]`

**Multiple cycles supported**: User can iterate as many times as needed.

---

## Board Mode Differences from Standalone

| Aspect | Standalone Mode | Board Mode |
|--------|----------------|------------|
| **Invocation** | User types `/research {topic}` | bb.exec reads board section `[ready]` |
| **Phase 1-2** | Interactive self-review loop | Fully autonomous (no user prompts) |
| **Gate 1** | Present synthesis, ask store/refine/discard | Auto-proceed (no gate) |
| **Gate 2** | Present reviewed draft, ask approve/edit/discard | Auto-proceed (no gate) |
| **Gate 3** | Ask user for placement info | Use board metadata (scope, parent, tags) |
| **Output** | Repository write only | Repository write + summary to board section |
| **Status tracking** | None | Update board section status tags throughout |
| **Refinement** | User re-invokes skill with new direction | User edits board section, re-runs bb.exec |
| **Scratch cleanup** | Always delete on completion | Delete on success, preserve on error |

---

## Status Tag Lifecycle

Board section status tag progresses through these states:

```
[pending] → User planning, not ready to execute
    ↓
[ready] or [ready:1] → User signals: "Execute now"
    ↓
[executing:1] → research updates at start
    ↓
[done:1] or [error:1] → research updates at end
    ↓
(if user wants refinement)
    ↓
[ready:2] → User edits Response section, re-runs
    ↓
[executing:2] → research updates at start (cycle 2)
    ↓
[done:2] or [error:2] → research updates at end
    ↓
(repeat as needed...)
```

**Tag format**:
- `[pending]` — Not ready (no cycle number)
- `[ready]` or `[ready:N]` — Ready to execute (N = cycle number)
- `[executing:N]` — Currently running
- `[done:N]` — Successfully completed
- `[error:N]` — Failed with error

---

## Error Handling in Board Mode

| Scenario | Behavior |
|----------|----------|
| Board section missing required fields | Update status to `[error:N]`, write error message to Cycle N Results, stop |
| Both searches fail (no results) | Update status to `[error:N]`, write error + suggestions to board, stop |
| Scope path not found | Update status to `[error:N]`, ask user to fix scope ID in board section, stop |
| Repository write fails | Update status to `[error:N]`, preserve scratch space, report failure to board |
| Narrative reviewer unavailable | Log warning, proceed with unreviewed draft |
| Single search fails | Continue with other search results, note in summary |
| Some page reads fail | Continue with successful reads, note in summary |
| Scratch space conflicts with scope folder | Update status to `[error:N]`, report path error, stop |

**Error message format in board section**:

```markdown
### Cycle 1 Results
❌ Research failed

**Error**: Both web search and internal search returned no results.

**Suggestions**:
- Try more specific keywords
- Check if topic is too niche or too broad
- Provide URLs in Response section for manual source addition

**Next**: Update Response section with refined query or manual sources, then re-run.
```

---

## Board Section Parsing

When invoked by bb.exec, research must parse board section to extract:

### Required Fields

**Topic** (from Recommendations):
```markdown
- Topic: "Compare Amazon Bedrock Agents and LangGraph"
```

**Scope** (from Recommendations):
```markdown
- Scope: S05_Technology
```

If these are missing → Error, update status to `[error:N]`, stop.

### Optional Fields

**Output format** (from Recommendations):
```markdown
- Output format: findings | six-pager | white-paper
```
Default: `findings` if not specified

**Parent** (from Recommendations):
```markdown
- Parent: P05_AI_Systems
```
Default: None (no parent backlink)

**Tags** (from Recommendations):
```markdown
- Tags: agents, bedrock, langgraph, comparison
```
Default: Auto-derive from topic

**User direction** (from Response):
```markdown
### Response
Focus on cost comparison and enterprise features.
```
Default: None (standard broad search)

**Cycle number** (from status tag):
```markdown
## 🔍 Research: {Topic} [ready:2]
```
Default: 1 if not specified

---

## Integration with bb.exec

bb.exec (Blackboard executor skill) handles:

1. Reading board file
2. Identifying sections marked `[ready]`
3. Invoking appropriate skill based on section emoji/type
4. Passing board context to skill
5. Monitoring status tag updates
6. Logging execution results

**research** handles:

1. Accepting board context from bb.exec
2. Updating status tags throughout execution
3. Autonomous Phase 1-3 execution (no interactive gates)
4. Writing summary to board section
5. Writing full results to repository
6. Returning completion status to bb.exec

**For bb.exec implementation details**, see: `.shared/skills/bb.exec/SKILL.md`

---

## Future Enhancements

Potential improvements to board integration:

**Scheduled research**:
- Board section with `[scheduled:YYYY-MM-DD]` tag
- bb.exec runs research on specified date
- Useful for periodic competitive analysis

**Multi-topic research**:
- Board section lists multiple related topics
- research runs each, synthesizes across all
- Single findings document covering all topics

**Confidence thresholds**:
- Board metadata: `- Min confidence: High`
- If research returns Medium or Low, auto-trigger refinement
- Only mark `[done]` when confidence threshold met

**Source constraints**:
- Board metadata: `- Max sources: 20` or `- Source types: academic, vendor`
- Control scope and depth of research

**Integration with other skills**:
- research findings feed into narrative skills (PRFAQ, six-pager)
- Board section chains: Research → PRFAQ → Review

---

## Notes

- This file contains **board mode integration only**
- For standalone mode, see main `SKILL.md` and other references
- For Phase 1-2 (search, synthesis), see `search-synthesis.md`
- For Phase 3 (document generation), see `document-generation.md`
- For bb.exec implementation, see `.shared/skills/bb.exec/SKILL.md`
