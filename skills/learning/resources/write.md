# write

Generate three writing versions from vault insights.

## Usage

- `/learning write` (auto-detect topic from today)
- `/learning write [topic]` (explicit topic)

## Six-Step Workflow

### Step 1: Identify Topic

Accept explicit topic or extract most compelling insight from today's daily note (or past 3 days if sparse).

### Step 2: Mine Supporting Material

Search vault systematically:
```bash
obsidian search query="[topic]"
obsidian backlinks path="[note].md"
```

Assess temporal velocity (accelerating/steady/decelerating) to gauge readiness.

### Step 3: External Research

Gather:
- Scientific research
- Frameworks
- Historical examples
- Counterintuitive findings

### Step 4: Identify Non-Obvious Angle

What would surprise readers?
Where does lived experience diverge from conventional wisdom?
Internal contradictions within vault evidence?

### Step 5: Generate Three Versions

**Short Post (1-3 paragraphs):**
- Open with sharpest insight
- Grounded in concrete detail

**Personal Essay (300-600 words):**
- Flow from specific triggering moment
- Through examples
- To larger connection
- End with open question

**Universal Essay (800-1500 words):**
- Written for specific stuck reader
- Diagnosis-then-reframe structure
- No neat closure

All: direct voice, first or second person, avoid preachiness.

### Step 6: Solicit Feedback

Ask which version resonates, what details need adjustment, whether to publish or save as draft.

Use `obsidian` CLI for searches.
