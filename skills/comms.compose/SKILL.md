---
name: comms.compose
description: Draft and iterate on professional correspondence (email, Slack, or other channels) using file-based workspace with conversation tracking
---

# Compose — Draft, Iterate, Log

Draft responses to emails, Slack messages, MBR write-ups, and other professional correspondence. Works with structured Scriptorium files that contain the user's ask, context, and a running conversation log.

## Invocation

Call this skill when the user requests:
- "Draft a response to [email/Slack/message]"
- "Compose a new [email/Slack message]"
- "Help me write [correspondence]"
- "Create a new correspondence file"
- `/comms.compose [optional: new|process|help]`

## Core Principles

- **Recommend-first.** Generate options or drafts for review. Never finalize without user approval.
- **Voice preservation.** Match the user's natural communication style — direct, warm, concise. No corporate template voice.
- **File-as-workspace.** The correspondence file is the single artifact. Everything lives in one place.
- **Iteration tracking.** Every round of feedback is logged in the Conversation Log section.

## Help

If the user's input is the word "help" (case-insensitive), skip all workflows and respond with:

```
✍️ Compose — Quick Reference

Two modes for drafting professional correspondence.

Usage:
  • Process mode: open a Scriptorium file, invoke /comms.compose
  • New mode: invoke /comms.compose new to create a file from scratch
  • Auto-detect: /comms.compose (uses active file if available)

File template sections:
  • ## Ask — what you need (your prompt to the assistant)
  • ## Context — background info, pasted messages, file references
  • ## Options — numbered choices (when you request multiple)
  • ## Response — the working draft
  • ## Conversation Log — iteration history (auto-maintained)

Workflow:
  1. You fill in Ask + Context (or the assistant walks you through it)
  2. Assistant generates options or a draft
  3. You provide feedback → assistant revises → logged
  4. Repeat until approved

Channels: Email, Slack, or any written correspondence
Files live in: Scriptorium/{Channel}/

Future: Channel-specific sending via MCP servers (not yet implemented)
```

Then stop — do not proceed to any workflow.

## Mode Detection

| Mode | Trigger | Behavior |
|------|---------|----------|
| `process` | User provides/has a file open with `## Ask` section | Read file, extract Ask + Context, generate response |
| `new` | User says "new", "create", "start fresh", or no valid file available | Walk user through file creation, then process |

**Detection logic**:
1. If user explicitly says "new" or "create" → **new mode**
2. If file path provided or active editor file exists:
   - If file has `## Ask` section → **process mode**
   - If file exists but no `## Ask` section → offer to help fill it in or switch to new mode
3. If no file available → **new mode**

## Requirements

**Tools needed**: Read, Write

**Future MCP servers** (not yet implemented):
- `aws-outlook-mcp` - For sending emails
- Slack MCP server - For posting messages
- Other channel-specific servers as needed

**Note**: Currently, this skill generates drafts for manual sending. Future versions may support channel-specific sending via MCP servers.

---

## Scriptorium File Template

### Naming Convention

```
{Topic}_{YYYYMMDD}.md
```

Use PascalCase or camelCase words separated by underscores. Date is the creation date.

**Examples**:
- `Customer_Escalation_Response_20260315.md`
- `Team_Update_Slack_20260315.md`
- `Leadership_MBR_Summary_20260315.md`

### Frontmatter

```yaml
---
aliases:
  - ""
creationDate: "{DayName} {Dth} {Month} {YYYY}"
timestamp: "{YYYYMMDDHHmmss}"
docType: correspondence
channel: "{email|slack|mbr|other}"
subType: "{channel}"
recipient: ""
subject: ""
status: draft
tags:
  - correspondence
  - {channel}
---
```

### Section Structure

```markdown
# {Subject}

## Ask

{What the user needs — the prompt/instruction to the assistant.
Can request multiple options or a single response.
Can specify tone, length, audience, or constraints.}

## Context

{Background information the assistant needs.
Can include pasted messages, file references, links, or free-form notes.
File references: `path/to/file.md` or [[wikilink]]}

## Options

{Numbered options when the Ask requests multiple choices.
Written by the assistant. User selects or mixes.
Omit this section if the Ask requests a single response.}

## Response

{The final or working draft. Written by the assistant.
Updated in place during iteration.}

## Conversation Log

{Chronological record of iterations. Newest entry at top.
Each entry: timestamp, summary, details.}
```

---

## Process Mode

Use when working with an existing Scriptorium correspondence file.

### Steps

1. **Identify the target file.**
   - If a file path was provided, use that file
   - If no file path was provided, use the currently active editor file
   - If no active file, ask the user to specify a file or switch to new mode

2. **Read and validate the file.**
   - Read the full file content
   - Verify it contains a `## Ask` section with content
   - If no Ask section or it's empty, notify the user: "This file doesn't have an Ask section. Want me to help you fill it in, or switch to new mode to create a fresh file?"

3. **Restore session context.**
   - Check for an existing `## Conversation Log` section
   - If present, read it to understand prior iterations and decisions
   - Acknowledge prior context briefly: "Picking up where we left off — last session you {summary}."

4. **Resolve file references in Context.**
   - Scan the `## Context` section for file references:
     - Backtick paths: `` `path/to/file.md` `` → read the file for additional context
     - Wikilinks: `[[filename]]` → resolve within workspace and read
   - Use referenced file content to inform the response. Do not modify referenced files.

5. **Generate response.**
   - Read the Ask carefully. Determine whether it requests:
     - Multiple options → write numbered options to `## Options`
     - A single response → write draft to `## Response`
   - Apply tone and voice rules (see below)
   - Preserve all existing file content — frontmatter, Ask, Context, and any user notes

6. **Present summary to user.**
   - After writing to the file, provide a brief summary in chat:
     - What you generated (e.g., "Wrote 3 options into the Options section")
     - Any assumptions you made
     - Ask if they want to refine, select an option, or approve

7. **Log the interaction.**
   - Append an entry to `## Conversation Log` (newest first):
     ```markdown
     ### {YYYY-MM-DD HH:MM} — Initial draft generated

     Generated {n options / a draft response} based on the Ask.
     {Note any key assumptions or decisions.}
     ```

---

## New Mode

Use when creating a new correspondence file from scratch.

### Steps

1. **Gather file metadata.** Ask the user (batched, max 3 questions):
   - What channel is this for? (Email, Slack, or describe it)
   - Who's the audience or recipient?
   - What's the subject or purpose? (one line)

2. **Create the file.**
   - Determine the subfolder: `Scriptorium/{Channel}/` (capitalize channel name)
   - Generate filename: `{Subject}_{YYYYMMDD}.md` using today's date
   - Create the file with frontmatter and empty section headers from the template

3. **Gather the Ask.**
   - Ask the user: "What do you need? Describe what you want the response to accomplish — tone, length, number of options, any constraints."
   - Write their input into the `## Ask` section

4. **Gather the Context.**
   - Ask the user: "What background does the assistant need? Paste the message you're responding to, reference files, or describe the situation."
   - Write their input into the `## Context` section

5. **Proceed to process mode.** Continue from Process Mode step 3 (restore session context → generate response)

---

## Iteration Flow

When the user provides feedback on a generated draft or options:

1. **Read the feedback.** Understand what needs to change — specific edits, tone adjustments, option selection, or a complete rewrite.

2. **Revise the file.**
   - If the user selected an option from `## Options`: copy the selected option (with any requested modifications) into `## Response`
   - If the user requested edits to `## Response`: update the Response section in place
   - If the user wants to combine options: merge as directed and write to `## Response`

3. **Log the iteration.** Append to `## Conversation Log`:
   ```markdown
   ### {YYYY-MM-DD HH:MM} — {Action summary}

   {What the user requested. What changed. Any decisions made.}
   ```

4. **Present the revision.** Briefly summarize what changed and ask if it's ready or needs more work.

5. **On approval.** Log the approval:
   ```markdown
   ### {YYYY-MM-DD HH:MM} — Approved

   Final response approved by user. {Any notes about where it was sent/posted.}
   ```
   Update frontmatter `status` to `complete` if the user confirms they're done.

---

## Tone and Voice Rules

All drafts follow the user's communication style:

- **Direct.** Get to the point in the first sentence. No throat-clearing.
- **Warm but not effusive.** Brief acknowledgment before substance. No over-the-top pleasantries.
- **Concise.** Every sentence earns its place. No filler paragraphs.
- **Channel-appropriate.** Slack is more casual than email. MBR summaries are tighter than Slack. Match the medium.
- **Audience-aware.** Leadership gets crisper language. Peers get more conversational tone. External contacts get slightly more formal.
- **Authentic.** Preserve the user's natural voice. Don't soften directness or add diplomatic padding unless the user asks for it.

**For detailed voice guidance**, see `references/tone-guide.md`.

---

## Channel-Specific Considerations

### Email
- Subject line matters
- Greeting and sign-off conventions
- Longer form acceptable
- Future: Send via `aws-outlook-mcp` MCP server

### Slack
- More casual tone
- Shorter paragraphs
- Emojis acceptable (sparingly)
- Thread-aware context
- Future: Post via Slack MCP server

### MBR / Reports
- Tightest writing
- Data-driven
- Executive summary style
- No pleasantries

### Other
- Match the context provided by user
- Ask about conventions if unclear

**For channel-specific details**, see `references/channel-specifics.md`.

---

## Error Handling

| Scenario | Behavior |
|----------|----------|
| No file specified and no active editor file | Ask user to specify a file path or switch to new mode |
| File exists but has no Ask section | Offer to help fill it in or switch to new mode |
| File reference in Context can't be resolved | Note the broken reference in chat, continue with available context |
| Scriptorium subfolder doesn't exist | Create it (e.g., `Scriptorium/Other/`) |
| User provides feedback but no draft exists yet | Generate the initial draft first, then apply feedback |

---

## Boundaries

- Never send messages on behalf of user (currently manual sending only)
- Always generate drafts for review, never finalize without approval
- Preserve all user-written content in files
- Never modify referenced files (read-only)
- Always log iterations in Conversation Log
- Match user's voice, don't impose corporate templates
- Ask clarifying questions rather than guess

---

## Future Capabilities

**Channel-specific sending** (not yet implemented):

When MCP servers become available, this skill may support:
- Email sending via `aws-outlook-mcp`
- Slack posting via Slack MCP server
- Other channel integrations

**Planned workflow**:
1. User approves draft in `## Response`
2. Skill asks: "Ready to send via [channel]?"
3. On confirmation, uses appropriate MCP server to send
4. Logs sent confirmation in Conversation Log
5. Updates frontmatter `status` to `sent`

Until then, users manually copy/paste drafts into their chosen channel.

---

## Implementation Notes

**Current state**: Draft generation and iteration only. User manually sends.

**File structure**: Uses Scriptorium workspace pattern for persistent iteration tracking.

**Mode flexibility**: Auto-detects from context, but user can force mode with explicit command.

**Voice preservation**: Critical to match user's authentic style, not generic AI writing.
