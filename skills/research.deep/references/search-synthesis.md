# Search & Synthesis Workflow

Complete Phase 1-2 logic for research.deep skill.

---

## Configuration (Scratch Space Resolution)

At the start of every research session, resolve the scratch space path before doing anything else.

### Steps

1. **Read config**: Read `config/config.json` and parse it as JSON.
   - If the file is missing, unreadable, or contains invalid JSON, set `scratchSpaceRoot` to `Tesseract/` and warn the user:
     > ⚠️ Could not read `config/config.json` — using default scratch space `Tesseract/`.
   - If the file is readable, continue to step 2.

2. **Extract scratch space path**: Look for the `researchScratchSpace` key in the parsed config object.
   - If the key exists and its value is a non-empty string, set `scratchSpaceRoot` to that value.
   - If the key is missing or its value is empty/null, set `scratchSpaceRoot` to `Tesseract/` and proceed normally (no warning needed — this is an expected default).

3. **Validate the path is not inside a scope folder**: Check whether any segment of the resolved `scratchSpaceRoot` path starts with `S` followed by one or more digits and then an underscore (the pattern `S{digits}_`, e.g. `S4061_AmazonSA`).
   - If the path is inside a scope folder (any path segment matches `S\d+_`), **reject the path**. Do not use it. Report the conflict to the user:
     > 🚫 The configured scratch space path `{path}` is inside a scope folder. Research scratch space must not overlap with scope folders. Please update the `researchScratchSpace` value in `config/config.json` and try again.
   - Then **stop** — do not proceed with the research session until the user updates the config.

4. **Ensure the directory exists**: If `scratchSpaceRoot` does not exist on disk, create it. This is a silent operation — no user confirmation needed for creating the scratch space directory.

### Summary

| Condition | Resolved path | User notification |
|-----------|---------------|-------------------|
| Config readable, key present, valid path | Value from config | None |
| Config readable, key missing or empty | `Tesseract/` | None |
| Config missing or unreadable | `Tesseract/` | Warning: config could not be read |
| Path is inside a scope folder (`S{digits}_`) | Rejected | Error: path conflicts with scope folder, stop session |

---

## Mode Detection & Session Setup

After resolving the scratch space path, parse the user's input and set up the research session.

### Step 1: Parse the research topic

The user's input (everything after the skill trigger) is the research topic. Trim leading and trailing whitespace.

- If the trimmed input is empty, ask the user to provide a topic and stop:
  > Please provide a research topic. For example: "Bedrock Agents vs LangGraph" or "EKS cost optimization strategies".
- If the trimmed input is the word `help` (case-insensitive), respond with the help text from the main SKILL.md and stop.
- Otherwise, the trimmed input is the **topic** for this session.

### Step 2: Generate the slug

Convert the topic into a kebab-case slug for the session directory name:

1. Convert to lowercase.
2. Replace any character that is not a letter, digit, or hyphen with a hyphen (`-`).
3. Collapse consecutive hyphens into a single hyphen.
4. Strip leading and trailing hyphens.
5. Truncate to 50 characters.
6. Strip any trailing hyphens introduced by truncation.

If the slug is empty after these steps (e.g. the topic was all special characters), use the fallback slug `research`.

**Examples:**

| Topic | Slug |
|-------|------|
| `Bedrock Agents vs LangGraph` | `bedrock-agents-vs-langgraph` |
| `EKS cost optimization strategies` | `eks-cost-optimization-strategies` |
| `What's new in Lambda@Edge?` | `what-s-new-in-lambda-edge` |
| `S3 → Glacier deep archive transition policies & best practices for long-term compliance` | `s3-glacier-deep-archive-transition-policies-best-p` (truncated to 50) |
| `!!!` | `research` (fallback) |

### Step 3: Build the session directory path

Construct the session directory path:

```
{scratchSpaceRoot}/{YYYYMMDD}-{slug}/
```

Where `{YYYYMMDD}` is today's date (e.g. `20260315`) and `{slug}` is the result from Step 2.

**Example:** `Tesseract/20260315-bedrock-agents-vs-langgraph/`

### Step 4: Handle duplicate directories

Check whether the session directory already exists on disk.

- If it does **not** exist, proceed to Step 5.
- If it **does** exist, append a numeric suffix starting at `-2` and increment until a non-existing path is found:
  - `{scratchSpaceRoot}/{YYYYMMDD}-{slug}-2/`
  - `{scratchSpaceRoot}/{YYYYMMDD}-{slug}-3/`
  - … and so on.

**Example:** If `Tesseract/20260315-bedrock-agents-vs-langgraph/` already exists, use `Tesseract/20260315-bedrock-agents-vs-langgraph-2/`.

### Step 5: Create the session directory

Create the session directory at the resolved path. This is a scratch space operation — no user confirmation needed.

### Step 6: Initialize `session.md`

Create `session.md` inside the session directory with the following content:

```markdown
---
topic: "{topic}"
started: "{YYYY-MM-DDTHH:mm}"
status: "in-progress"
sources_searched: 0
sources_read: 0
---
# Research Session: {Topic}

## Running Synthesis

{empty — will be populated during Phase 2}

## Sources Index

| # | Type | Title | URL | Note File |
|---|------|-------|-----|-----------|
```

Where:
- `{topic}` is the user's original input (trimmed, preserving original casing).
- `{YYYY-MM-DDTHH:mm}` is the current date and time in ISO format.
- `{Topic}` in the heading uses the original casing of the user's input.

### Step 7: Confirm session creation

Report the session setup to the user:

> 📂 Research session started.
> - **Topic:** {topic}
> - **Scratch space:** `{session directory path}`
>
> Proceeding to search public and internal sources…

Then continue to Phase 1 search step.

---

## Phase 1: Search, Gather & Self-Review

After the session directory and `session.md` are initialized, begin searching for sources on the user's topic.

### Step 1: Search

Search both public and internal sources in parallel. Both searches can be issued in the same turn — they are independent.

#### 1a. Search public sources

Use `remote_web_search` with the user's topic as the query.

```
remote_web_search({ query: "{topic}" })
```

- If the tool returns results, collect each result's **title**, **URL**, and **snippet** (used as the relevance description).
- If the tool fails or times out, log the error in `session.md` under a `## Errors` section at the bottom of the file:
  ```
  ## Errors
  - [{YYYY-MM-DD HH:mm}] `remote_web_search` failed: {error message}
  ```
  Report the failure to the user:
  > ⚠️ Public search failed: {error message}. Continuing with internal sources.

  Then continue — do not stop the session.

#### 1b. Search internal sources

Use `mcp_amzn_mcp_search_internal_websites` with the user's topic as the query.

```
mcp_amzn_mcp_search_internal_websites({ query: "{topic}" })
```

- If the tool returns results, collect each result's **title**, **URL**, and **snippet**.
- If the tool fails, log the error in `session.md` under `## Errors`:
  ```
  - [{YYYY-MM-DD HH:mm}] `mcp_amzn_mcp_search_internal_websites` failed: {error message}
  ```
  Report the failure to the user:
  > ⚠️ Internal search failed: {error message}. Continuing with public sources.

  Then continue with whatever results are available.

#### 1c. Handle total failure

If **both** search tools fail, report to the user and stop:

> 🚫 Both public and internal searches failed. Unable to gather sources for "{topic}".
>
> Suggestions:
> - Try again later (the search services may be temporarily unavailable)
> - Provide specific URLs to read directly

Then stop — do not proceed to Step 2.

#### 1d. Handle no results

If both tools succeed but return **zero** results combined, report to the user:

> No results found for "{topic}". Try refining the topic or using alternative keywords.

Then stop — do not proceed to Step 2.

#### 1e. Update session metadata

After both searches complete (or one fails and the other succeeds), update the `sources_searched` counter in `session.md` frontmatter to reflect the total number of results collected from both searches.

### Step 2: Read and Capture Notes

For each relevant result from Step 1, read the full page content and create a structured source note.

#### 2a. Read page content

Use the appropriate tool based on source type:

- **Public sources**: Use `webFetch` with the result URL.
  ```
  webFetch({ url: "{url}" })
  ```
- **Internal sources**: Use `mcp_amzn_mcp_read_internal_website` with the result URL.
  ```
  mcp_amzn_mcp_read_internal_website({ url: "{url}" })
  ```

Multiple page reads can be issued in parallel when they are independent of each other.

If a page read fails for a specific source:
- Skip that source.
- Note the failure in the Sources Index table in `session.md` by adding the row with `(failed)` in the Note File column.
- Log the error in `session.md` under `## Errors`:
  ```
  - [{YYYY-MM-DD HH:mm}] Failed to read {url}: {error message}
  ```
- Continue reading the remaining sources — do not stop.

#### 2b. Create source note files

For each successfully read source, create a note file in the session directory using sequential numbering:

**File path:** `{session directory}/N_source-{n}.md`

Where `{n}` starts at 1 and increments for each source read.

**File content:**

```markdown
# {Source Title}

- **URL**: {url}
- **Type**: public | internal
- **Retrieved**: {YYYY-MM-DD HH:mm}

## Summary

{2-3 sentence summary of the content that is relevant to the research topic}

## Key Points

- {bullet points of relevant information extracted from the page}

## Excerpts

> {direct quotes or key passages, with surrounding context for clarity}
```

Guidelines for note content:
- The **Summary** should focus on what is relevant to the research topic, not a generic page summary.
- **Key Points** should be specific, factual bullets — not vague restatements.
- **Excerpts** should be direct quotes (blockquoted) with enough context to be useful standalone. Keep excerpts concise — no more than 30 consecutive words from a single source to respect content licensing.

#### 2c. Update session.md

After creating each source note, update `session.md`:

1. **Frontmatter**: Increment the `sources_read` counter.
2. **Sources Index table**: Add a row for the source:

   ```
   | {n} | {public/internal} | {title} | {url} | N_source-{n}.md |
   ```

#### 2d. Present sources to user

> **Important:** Do NOT execute this step immediately after 2c. Proceed to **Step 3: Self-Review Loop** first. Return here only after the self-review loop is complete.

After the self-review loop (Step 3) is satisfied, present the collected sources to the user using this format:

```markdown
## 🔍 Sources Found — {topic}

### Public Sources ({count})
1. **{title}** — {url}
   {1-2 sentence relevance description}

### Internal Sources ({count})
1. **{title}** — {url}
   {1-2 sentence relevance description}

---
Focus on specific sources? Skip any? Or proceed with all?
```

Where:
- `{count}` is the number of successfully read sources of that type (including any added during self-review).
- If a type has zero sources, still show the section with `(0)` and the text "No {public/internal} sources found."
- The relevance description comes from the Summary in the source note (condensed to 1-2 sentences).

Then wait for user direction before proceeding to Phase 2.

### Step 3: Self-Review Loop

After all sources are read and notes captured (Step 2c), and BEFORE presenting sources to the user (Step 2d), enter an autonomous self-review loop. The user does not see anything until this loop completes.

This loop is fully autonomous — no user approval, no progress updates, no questions. Work silently in the scratch space until satisfied.

#### 3a. Review findings for completeness

Read all `N_source-{n}.md` files in the session directory and the Running Synthesis section of `session.md`. Evaluate the gathered material against these criteria:

1. **Coverage gaps**: Are there obvious subtopics, perspectives, or aspects of the research topic that no source addresses? For example, if the topic is "Bedrock Agents vs LangGraph," do the sources cover both technologies, or is one side missing?
2. **Undefined terms**: Are there acronyms, service names, or technical terms referenced in the sources that are not explained and would leave the synthesis unclear?
3. **Contradictions**: Do any sources directly contradict each other on factual claims? Note the specific conflict and which sources are involved.
4. **Unanswered key questions**: Based on the topic, are there questions a reader would naturally ask that the current sources do not answer?

Produce an internal assessment (do not write it to any file — hold it in context):

- List of identified gaps (if any)
- List of undefined terms needing clarification (if any)
- List of contradictions (if any)
- List of unanswered key questions (if any)

If all four lists are empty, the review is satisfied — skip to Step 3d.

#### 3b. Perform targeted searches to fill gaps

For each gap, undefined term, or unanswered question identified in 3a, perform targeted searches:

- Use `remote_web_search` and/or `mcp_amzn_mcp_search_internal_websites` with specific, narrowed queries derived from the gap. For example:
  - Coverage gap "no sources on LangGraph architecture" → search `"LangGraph architecture components"`
  - Undefined term "LCEL" → search `"LCEL LangChain Expression Language"`
  - Unanswered question "pricing comparison" → search `"Bedrock Agents pricing vs LangGraph hosting cost"`

- For contradictions, search for authoritative sources (official documentation, AWS docs) that can resolve the conflict.

- Read the most relevant results using `webFetch` (public) or `mcp_amzn_mcp_read_internal_website` (internal), following the same read logic as Step 2a.

- If a targeted search or page read fails, log the error in `session.md` under `## Errors` and move on to the next gap. Do not stop the loop for individual failures.

#### 3c. Create notes and update session.md

For each new source read during the targeted search:

1. Create a new `N_source-{n}.md` file in the session directory, continuing the sequential numbering from Step 2b. Use the same note structure (title, URL, type, retrieved date, summary, key points, excerpts).
2. Update `session.md`:
   - Increment the `sources_read` counter in frontmatter.
   - Add a row to the Sources Index table for the new source.

#### 3d. Evaluate and iterate

After filling gaps (or if no gaps were found in 3a), re-evaluate the full set of findings:

- Re-read all source notes (including any new ones from 3b/3c).
- Apply the same four criteria from 3a: coverage gaps, undefined terms, contradictions, unanswered key questions.
- If new gaps remain, loop back to 3b.

**Iteration cap:** Maximum 3 iterations of the self-review loop (3a → 3b → 3c → 3d counts as one iteration). After 3 iterations, proceed to Step 2d regardless of remaining gaps. If gaps still exist after 3 iterations, note them internally — they will surface as "Open Questions" during synthesis in Phase 2.

#### 3e. Proceed to source presentation

When the self-review loop is satisfied (all four criteria are clear, or the iteration cap is reached), proceed to **Step 2d** to present the full set of sources to the user.

### Step 4: User Direction

After Step 2d presents the source list and asks the user for direction, **wait for the user's response**. Do not proceed to Phase 2 until the user responds.

Parse the user's response and take the corresponding action:

#### 4a. Proceed with all sources

If the user responds with an affirmative — e.g. "proceed", "all", "looks good", "continue", "yes", "go ahead", "lgtm", or similar — proceed to **Phase 2** with all sources marked as active.

No changes to `session.md` or source notes are needed.

#### 4b. Focus on specific sources

If the user responds with a focus instruction — e.g. "focus on 1, 3, and 5", "only sources 1 and 4", "just 2, 3", or similar — extract the source numbers from the response.

1. Mark only the specified sources as **active** for synthesis. All other sources are **deprioritized** — they remain in the scratch space and Sources Index but are excluded from the Running Synthesis and any subsequent Phase 2 output.
2. Update `session.md` by adding a `focus` field to the frontmatter listing the active source numbers:
   ```yaml
   focus: [1, 3, 5]
   ```
3. Do not delete or modify the deprioritized source note files — they stay in the session directory in case the user changes their mind later.
4. Proceed to **Phase 2** using only the active sources.

#### 4c. Skip specific sources

If the user responds with a skip instruction — e.g. "skip 2", "skip sources 2 and 4", "drop 3", "exclude 1", or similar — extract the source numbers from the response.

1. Mark the specified sources as **skipped**. All remaining sources are active for synthesis.
2. Update `session.md` by adding a `skipped` field to the frontmatter listing the skipped source numbers:
   ```yaml
   skipped: [2, 4]
   ```
3. Do not delete or modify the skipped source note files — they stay in the session directory.
4. Proceed to **Phase 2** using only the non-skipped sources.

#### 4d. Add a new source

If the user responds with a URL to add — e.g. "add https://docs.aws.amazon.com/...", "also read https://...", "include this: {url}" — extract the URL from the response.

1. Read the URL using the appropriate tool:
   - If the URL matches an internal Amazon domain, use `mcp_amzn_mcp_read_internal_website`.
   - Otherwise, use `webFetch`.
2. Create a new `N_source-{n}.md` file in the session directory, continuing the sequential numbering. Use the same note structure as Step 2b.
3. Update `session.md`:
   - Increment the `sources_read` counter in frontmatter.
   - Add a row to the Sources Index table for the new source.
4. Re-present the updated source list to the user using the same format as Step 2d, then **wait again** for user direction (loop back to the top of Step 4).

#### 4e. Refinement or new direction

If the user's response does not match any of the patterns above — e.g. "also look into cost comparisons", "can you find more about the security model?", "what about performance benchmarks?" — treat it as a refinement request.

1. Use the user's response as additional search context. Loop back to **Step 1** (Search) and perform new searches combining the original topic with the user's refinement input.
2. Follow the same flow: search → read → create notes → self-review loop → present updated sources (Step 2d) → wait for user direction (Step 4 again).
3. New sources are appended to the existing session — do not discard previously gathered sources unless the user explicitly asks.

---

## Phase 2: Synthesize

After the user responds in Step 4 (Phase 1) and directs you to proceed, build the running synthesis in the scratch space and present it for review.

### Step 5: Build the Running Synthesis

Read all active source notes and synthesize them into a coherent summary in `session.md`.

#### 5a. Determine active sources

Read `session.md` frontmatter to check for `focus` or `skipped` fields set during Step 4:

- If `focus` is present, only source notes whose numbers appear in the `focus` list are active.
- If `skipped` is present, all source notes except those whose numbers appear in the `skipped` list are active.
- If neither field is present, all source notes are active.

Read every active `N_source-{n}.md` file in the session directory.

#### 5b. Identify themes and patterns

Analyze the active source notes and identify:

1. **Key themes**: Recurring topics, concepts, or arguments that appear across multiple sources. Group related findings under descriptive theme headings (e.g. "Architecture Differences", "Cost Considerations", "Security Model").
2. **Patterns**: Common recommendations, best practices, or conclusions that multiple sources agree on.
3. **Insights**: Non-obvious connections or implications that emerge from reading the sources together — things no single source states explicitly but that become apparent in combination.

Organize findings **by theme or subtopic, not by source**. Each theme should draw from whichever sources are relevant, citing source numbers inline (e.g. `[1, 3]`).

#### 5c. Note contradictions

Identify any points where sources directly disagree on factual claims, recommendations, or conclusions. For each contradiction:

- State the conflicting positions clearly.
- Cite which sources hold each position (by source number).
- If one source is more authoritative (e.g. official AWS documentation vs. a blog post), note that.

#### 5d. Identify open questions

List questions that the active sources do not fully answer. These may include:

- Gaps identified during the self-review loop (Step 3) that were not resolved within the iteration cap.
- Questions that naturally arise from the synthesized themes but are not addressed by any source.
- Areas where the user might want deeper investigation.

#### 5e. Write the Running Synthesis in session.md

Replace the `## Running Synthesis` section in `session.md` with the synthesized content. Use this structure:

```markdown
## Running Synthesis

### {Theme 1 Title}

{Synthesized findings for this theme, drawing from relevant sources. Cite source numbers inline, e.g. [1, 3].}

### {Theme 2 Title}

{Synthesized findings for this theme.}

{... additional themes as needed ...}

### Contradictions

{List of contradictions between sources, or "No contradictions identified." if none.}

- **{Point of disagreement}**: {Source X position} [X] vs. {Source Y position} [Y]. {Note on relative authority if applicable.}

### Open Questions

{List of unresolved questions, or "No open questions." if none.}

- {Question 1}
- {Question 2}
```

#### 5f. Update session status

Update the `session.md` frontmatter `status` field from `"in-progress"` to `"ready"`:

```yaml
status: "ready"
```

### Step 6: Present Synthesis to User (Gate 1)

After writing the Running Synthesis and updating the session status, present a clean summary to the user. Do NOT dump the raw `session.md` content — present a polished, readable summary.

Use this format:

```markdown
## 📋 Research Synthesis — {topic}

{High-level overview: 2-3 sentences summarizing the overall landscape of findings.}

### Key Findings

#### {Theme 1 Title}
{Concise summary of this theme's findings — 2-4 sentences. Cite source numbers inline [1, 3].}

#### {Theme 2 Title}
{Concise summary of this theme's findings.}

{... additional themes ...}

### Contradictions

{If contradictions exist:}
- **{Point}**: {Source X} says {position} [X], while {Source Y} says {position} [Y].

{If no contradictions:}
No contradictions identified across sources.

### Open Questions

{If open questions exist:}
- {Question 1}
- {Question 2}

{If no open questions:}
No open questions — sources provided comprehensive coverage.

### Sources Summary

{N} sources reviewed ({P} public, {I} internal).

---

**What would you like to do?**
- **Store** — choose an output format and generate a document
- **Refine** — adjust the research (add sources, dig deeper on a theme, re-search)
- **Discard** — delete the scratch space and end the session
```

Where:
- `{N}` is the total number of active sources used in the synthesis.
- `{P}` is the count of active public sources.
- `{I}` is the count of active internal sources.

Then **wait for the user's response**. Do not proceed until the user chooses store, refine, or discard. The user's response is handled by the Gate 1 approval logic (next step).

### Step 7: Gate 1 — User Decision

After Step 6 presents the synthesis and the store/refine/discard prompt, **wait for the user's response**. Do not proceed until the user responds. Parse the user's response and take the corresponding action:

#### 7a. Store

If the user responds with a store intent — e.g. "store", "save", "keep", "generate", "write it up", or similar — proceed to collect the output format.

Present the format choices:

> Which output format?
> - **findings** — structured research document (default)
> - **six-pager** — Amazon narrative format
> - **white-paper** — longer-form deep dive

Then **wait for the user's response**.

Parse the user's format choice:

- `"findings"`, `"default"`, or no explicit choice → format is `findings`
- `"six-pager"`, `"six pager"`, `"6-pager"`, `"narrative"` → format is `six-pager`
- `"white-paper"`, `"white paper"`, `"whitepaper"`, `"deep dive"` → format is `white-paper`
- If the user provides an unrecognized format, report available formats and ask again:
  > That format isn't supported. Available formats: **findings** (default), **six-pager**, **white-paper**.

Once a valid format is selected:

1. Update `session.md` frontmatter — add the `outputFormat` field:
   ```yaml
   outputFormat: "{format}"
   ```
   Where `{format}` is one of `findings`, `six-pager`, or `white-paper`.

2. Update `session.md` frontmatter `status` from `"ready"` to `"storing"`:
   ```yaml
   status: "storing"
   ```

3. Proceed to **Phase 3** (document generation, narrative review, placement, and repository write).

#### 7b. Refine

If the user responds with a refinement intent — e.g. "refine", "dig deeper", "more on X", "add sources", "can you also look into…", or provides specific direction on what to explore further — treat it as a refinement request.

1. Parse the user's refinement direction. This may be:
   - A request to explore a specific subtopic ("dig deeper on the security model")
   - A request for more sources ("add more internal sources")
   - A request to re-search with different terms ("also search for cost comparison")
   - General dissatisfaction ("needs more depth")

2. Preserve all existing sources and notes in the scratch space — do not delete or overwrite any `N_source-{n}.md` files or the current Running Synthesis.

3. Loop back to **Phase 1 Step 1** (Search) with the user's refinement direction as additional search context. Combine the original topic with the refinement input when constructing search queries.

4. Follow the full Phase 1 flow: search → read → create new source notes (continuing the sequential `N_source-{n}.md` numbering) → self-review loop → update `session.md` sources index.

5. After the new search/read/self-review cycle completes, rebuild the Running Synthesis in `session.md` incorporating both the original and new sources.

6. Return to **Step 6** — present the updated synthesis to the user with the same store/refine/discard prompt. The user may refine again (looping back here) or choose to store or discard.

#### 7c. Discard

If the user responds with a discard intent — e.g. "discard", "delete", "cancel", "nevermind", "drop it", or similar — ask for confirmation before deleting.

Present the confirmation prompt:

> Delete the scratch space for this session? This cannot be undone. (yes/no)

Then **wait for the user's response**.

- If the user confirms — e.g. "yes", "y", "confirm", "do it" — delete the entire session directory (the `{scratchSpaceRoot}/{YYYYMMDD}-{slug}/` folder and all its contents). Then report:
  > Research session discarded.

  The session is complete. Do not proceed to any further steps.

- If the user does not confirm — e.g. "no", "n", "wait", "actually…" — return to the store/refine/discard prompt from Step 6. Present it again:
  > No problem. What would you like to do?
  > - **Store** — choose an output format and generate a document
  > - **Refine** — adjust the research (add sources, dig deeper on a theme, re-search)
  > - **Discard** — delete the scratch space and end the session

  Then **wait for the user's response** and re-enter Step 7 parsing.

---

**Phase 3 continues in `document-generation.md`**
