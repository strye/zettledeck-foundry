# suggest-topics

Suggest article topics from vault analysis.

## Core Function

Scan vault for topics with:
- Graph density (well-developed)
- Current energy (recent activity)
- Synthesis potential (ready to be written)

## Implementation Approach

### Step 1: Identify Well-Developed Topics

**Criteria:**
- 10+ notes related to topic
- 5+ backlinks to central concept
- Distributed across multiple note types (diary, projects, content)

**Method:**
```bash
obsidian properties list  # Find top tags
obsidian search query="[topic]"  # Count mentions
obsidian backlinks path="[hub-note].md"  # Check density
```

### Step 2: Identify "Alive" Topics

**Recent activity indicators:**
- Mentioned in past 2 weeks of diary
- Recent note creations/edits on topic
- Appears in active projects
- Recent calendar time spent on topic

**Method:**
- Review recent diary entries for topic frequency
- Check file modification dates
- Cross-reference with active project lists

### Step 3: Score Synthesis Potential

**Factors:**
- Has clear thesis or question?
- Contains contradictions worth exploring?
- Evidence of evolution (thinking changed over time)?
- External resonance (others asking about this)?
- Personal energy (excitement evident in notes)?

**Scoring (1-5 each):**
- Development: How fleshed out?
- Energy: How alive right now?
- Novelty: How original?
- Clarity: How ready to articulate?
- Urgency: How timely?

**Total score:** Sum (max 25)

### Step 4: Suggest Topics

**For each high-scoring topic (15+):**
- Topic name
- Score breakdown
- Angle suggestion (what makes it interesting?)
- Vault evidence (key notes, quotes)
- Why now? (what makes this timely?)
- Target audience (who needs this?)

### Step 5: Integration with vault.map-topology

Could use topology analysis to find:
- Well-connected clusters ready for synthesis
- Bridge notes that connect multiple themes
- Topics at intersection of multiple domains

## Output Format

### Article Topic Recommendations

**Topic 1: [Name]**
- **Score:** [X]/25 (Development: [X], Energy: [X], Novelty: [X], Clarity: [X], Urgency: [X])
- **Angle:** [What makes this interesting?]
- **Vault Evidence:**
  - [[Key Note 1]] - [Quote or summary]
  - [[Key Note 2]] - [Quote or summary]
  - [X] mentions in past month
  - Last worked on: [Date]
- **Why Now:** [What makes this timely?]
- **Target Audience:** [Who needs this?]
- **Estimated Length:** [Short post / Essay / Long-form]

[Continue for top 3-5 topics]

## Constraints

- Minimum 10 related notes for topic to qualify
- Must have recent activity (past 30 days)
- Score must be 15+ to recommend
- Cap at 5 recommendations maximum
- All claims must cite specific vault notes

Use `obsidian` CLI for vault analysis.
