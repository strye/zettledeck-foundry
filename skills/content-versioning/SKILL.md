---
name: content-versioning
description: Content versioning workflow — source protection, working copies, version numbering, promotion process, changelog format, and editing practices. Activated when editing, revising, iterating on, or reviewing content files.
user-invocable: false
version: 1.0.0
tags: [content, versioning, workflow, editing]
---

# Content Versioning Workflow

Background knowledge for content editing operations. Auto-loaded when editing, revising, iterating on, or reviewing content files.

## Core Rule

The source document is never modified during iteration. It represents the last stable, approved version. All work happens in a `_versions/` subfolder within the content's directory until explicitly promoted.

## Working Copy Naming

Convention inside `_versions/`:

```
{original-filename}-v{version}.md
```

Example: `getting-started-v1.1.md`

## Version Numbers

Use two-part versioning: `{major}.{minor}`

| Change Type | Version Bump |
|-------------|--------------|
| Typo, broken link, minor formatting fix | No bump — note in changelog as patch |
| Section rewrite, new content, restructuring | Minor bump (1.0 → 1.1) |
| Document purpose, audience, or structure fundamentally changes | Major bump (1.x → 2.0) |

## Promotion

When a working copy is approved:

1. Copy its content back to the source file.
2. Update the source file's frontmatter `version` and `status` fields to match.
3. Leave `_versions/` intact as a full audit trail.

## Changelog Format

Each content document maintains a changelog at:

```
{content-folder}/_versions/{title}-CHANGELOG.md
```

Entry format (newest first):

```markdown
# Changelog: {Document Title}

Source: `../{source-filename}.md`

---

## [v1.1] - YYYY-MM-DD

**Status:** in-review
**Summary:** Brief description of what changed and why.

### Changes
- Specific change
- Specific change

### Notes
Context, open questions, or decisions that should carry forward to future sessions.

---

## [v1.0] - YYYY-MM-DD

**Status:** published
**Summary:** Initial version.
```

- Always prepend new entries at the top (newest first).
- At the start of any session on a content document, read the changelog before making changes to restore prior context.

## Editing Practices

- **Preserve the author's voice** and terminology unless the user explicitly directs a different style or voice. This is the default — the system does not "improve" or normalize an author's natural writing style.
- Do not reformat markdown structure (heading levels, list styles) unless that is the stated task.
- For any substantive edit, create a working copy in `_versions/` rather than editing the source in place.
- For minor corrections (typos, broken links), in-place edits are acceptable — note them in the changelog as a patch with no version bump.
- Read the full document and its changelog before making changes.
- Confirm the task intent and whether versioned revision is needed before creating new versions or modifying files. Default to asking rather than assuming edits don't need versioning.
