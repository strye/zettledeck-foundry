# Channel-Specific Guidelines

Detailed formatting, convention, and feature guidance for each communication channel.

## Email

### Structure
```
[Subject Line]

[Greeting]

[Body - 2-4 paragraphs max]

[Closing]
[Name]
```

### Subject Lines
- **Be specific.** "Q2 Roadmap Review" not "Meeting Follow-up"
- **Front-load keywords.** Put important words first for mobile previews
- **No all-caps.** Use sentence case
- **Keep under 60 characters** when possible

**Examples**:
- ✅ "API rate limit issue—need input by Friday"
- ✅ "Feedback on PRFAQ draft (attached)"
- ❌ "Quick question"
- ❌ "URGENT: PLEASE READ"

### Greetings

**Internal (Amazon):**
- Direct: No greeting, jump right in (acceptable for peers)
- Standard: "Hi [Name]," or "[Name]—"
- Formal: "Hi [Name]," (use for leadership or first contact)

**External:**
- Always use greeting: "Hi [Name]," or "Hello [Name],"
- First contact with customer: "Hi [Name],"

**Group emails:**
- "Hi team," or "Hi everyone," or "Team—"

### Sign-offs

**Internal:**
- Casual: No sign-off (just name)
- Standard: "Thanks," or "Best,"
- Never: "Warm regards," "Kind regards," "Sincerely" (too formal for internal)

**External:**
- Standard: "Best," or "Thanks,"
- Customer-facing: "Best," or "Thank you,"

### Formatting
- **Paragraphs:** 2-4 sentences max
- **Line breaks:** Blank line between paragraphs
- **Lists:** Use bullets or numbers for 3+ items
- **Bold:** Sparingly, for key decisions or action items
- **Links:** Use descriptive text, not raw URLs

### Future: Sending via MCP
When `aws-outlook-mcp` server is available:
- Compose draft in `## Response` section
- User approves
- Skill calls `send_email` tool with:
  - `to`: recipient(s)
  - `subject`: from frontmatter
  - `body`: from `## Response`
  - `cc`: if specified
  - `attachments`: if referenced

---

## Slack

### Structure
```
[Opening line - get to point]

[Body - 1-2 sentence paragraphs with line breaks]

[Closing - optional]
```

### Tone
- **More casual than email.** Conversational, like talking in person.
- **No greeting.** Jump right in.
- **No sign-off.** Just end when you're done.

### Formatting
- **Short paragraphs.** 1-2 sentences, then line break.
- **Line breaks liberally.** Use whitespace for readability.
- **Code blocks:** Use ``` for code, commands, or structured data
- **Bold:** Use `*bold*` for emphasis
- **Lists:** Use simple dashes or numbered lists

**Example**:
```
Quick update on the migration:

Batch 1 (20 accounts) completed yesterday. No issues.

Batch 2 starts tomorrow. Slightly riskier since they're production accounts—planning to do it during the maintenance window.

Let me know if you want to review the runbook first.
```

### Emojis
- **Sparingly.** Only if it adds clarity or appropriate levity.
- **Acceptable:** ✅ ❌ 🚀 👍 🔥 (reaction-style)
- **Avoid:** 😊 🙏 💯 (too casual/effusive for most contexts)
- **Never in serious/sensitive topics**

### Threads vs Direct Messages
- **Threads:** Keep related discussion together. Reference parent message.
- **DMs:** More direct, skip even more formality.

### Future: Posting via MCP
When Slack MCP server is available:
- Compose draft in `## Response` section
- User approves
- Skill calls Slack posting tool with:
  - `channel`: from frontmatter or user specifies
  - `message`: from `## Response`
  - `thread_ts`: if replying in thread

---

## MBR / Written Reports

### Structure
```
[Executive Summary - 2-3 sentences]

[Body - bullets or very short paragraphs]

[Data/Evidence - tables, metrics, links]
```

### Tone
- **Tightest writing.** Every word earns its place.
- **No pleasantries.** Pure substance.
- **Data-driven.** Back every assertion.
- **Bottom line first.** Decision → rationale → evidence.

### Formatting
- **Executive summary:** Lead with the conclusion/decision
- **Bullets preferred.** Dense paragraphs discouraged.
- **Headings:** Use ## for sections
- **Tables:** For comparative data
- **Bold:** For key metrics or decisions

**Example structure:**
```markdown
# Q2 Migration Status

**Summary:** On track for June 30 completion. Two risks identified.

## Progress
- 60% complete (120/200 accounts migrated)
- Zero downtime incidents
- Customer satisfaction: 4.8/5

## Risks
1. **API rate limits** - Mitigation: batching + caching
2. **Resource constraints** - Mitigation: additional engineers approved

## Next 30 Days
- Complete batch 3-5 (80 accounts)
- Cutover planning for high-priority customers
- Runbook finalization
```

### What to Cut
- Greetings/sign-offs
- Transition phrases ("Moving on to...")
- Hedging language ("We believe that possibly...")
- Unnecessary context (assume reader knows basics)

---

## Other Channels

### LinkedIn
- More formal than Slack, less formal than traditional email
- Professional but personable
- Shorter than email, longer than Slack
- Can use light emojis (professional ones only)

### SMS / Text
- Ultra-brief
- Assume limited context
- Link to longer content if needed
- Emojis acceptable

### Internal Wikis / Confluence
- Similar to MBR style
- Heavy use of headings, bullets, tables
- Link liberally to related docs
- Update timestamps and version info

---

## Cross-Channel Principles

### Regardless of channel:
1. **Get to the point quickly**
2. **Match the formality to the medium**
3. **Use formatting to aid scanning**
4. **Preserve the user's voice**
5. **Respect the reader's time**

### Adapt based on:
- **Urgency:** More direct when urgent
- **Audience:** More context for external, less for internal
- **Topic:** More formal for sensitive topics
- **Relationship:** More casual with close colleagues

---

## Future MCP Server Integration

### Planned Channel Support

**Email (aws-outlook-mcp)**:
- Send email with subject, body, recipients
- Reply to thread
- Forward with context
- Schedule send (if supported)

**Slack (future MCP server)**:
- Post to channel
- Reply in thread
- Direct message
- Edit posted message (if within time window)

**Not yet planned**:
- LinkedIn posting
- SMS sending
- Twitter/X posting
- Teams posting (may use similar to Slack)

### Workflow for Future Sending

1. User composes draft using this skill
2. Draft written to `## Response` section
3. User reviews and approves
4. Skill asks: "Ready to send via [channel]?"
5. On confirmation:
   - Skill calls appropriate MCP server tool
   - Logs send confirmation in Conversation Log
   - Updates frontmatter `status: sent`
   - Includes sent timestamp and any message IDs

Until MCP servers are integrated, all sending remains manual (copy/paste).
