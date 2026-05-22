---
name: coding-session-summary
description: Summarize long coding sessions into archival Markdown notes in an Obsidian vault. Use when the user asks to capture, archive, summarize, hand off, or preserve a coding session across hours or days.
---

# Coding Session Summary

Create a factual Markdown summary of a coding session and store it in the user's Obsidian vault for later retrieval.

## Output Location

Save the completed note under:

```text
~/current-obsidian-vault/Daily Notes
```

Use this filename format:

```text
<project-folder-name>-YYMMDDHHMM.md
```

Derive `<project-folder-name>` from the current workspace/repository directory name. Use the local current time for `YYMMDDHHMM`. If the workspace is not inside a project folder, use the most specific meaningful directory name available.

## Frontmatter

Do not use an Obsidian template. Start the note with this filled-out YAML frontmatter:

```markdown
---
project: <project-folder-name>
status: active
created: YYYY-MM-DD HH:MM
type: coding-session
hostname: <hostname>
---
```

Use the same project folder name used in the filename for `project`. Use local time for `created`. Get `hostname` from the current PC/Mac, for example with `hostname` on macOS/Linux. Keep `type` exactly `coding-session`.

## Summary Requirements

Summarize the coding session using the available conversation history, terminal output, code changes, commits, plans, errors, and any relevant context from the session.

The summary should be factual, concise, and useful for future reference. Do not include generic commentary or speculation. If something is uncertain, say so clearly.

Structure the document as follows after the frontmatter:

```markdown
# Coding Session Summary

## Highlights

- Implemented authentication flow for the dashboard.
- Fixed failing integration tests around session expiry.
- Refactored API client error handling to use typed result objects.
- Left one unresolved issue around token refresh retries.

## Detailed Summary

Write a series of coherent paragraphs explaining the session in chronological or logical order.
```

Highlights should be a short bulleted list of the most important outcomes from the session. Include only high-signal items, such as major features implemented, bugs fixed, architectural decisions, blockers resolved, or remaining risks.

The detailed summary should read like an engineering handoff note, not a transcript. Group related work together, avoid excessive detail, and preserve enough context that someone can resume the work later without rereading the full session.

Cover the following where relevant:

- The original goal or problem being worked on.
- Important investigation steps and findings.
- Code areas, files, modules, or systems touched.
- Key implementation decisions and why they were made.
- Bugs, errors, failed approaches, or debugging discoveries.
- Tests run, results observed, and any verification performed.
- Changes that were planned but not completed.
- Open questions, risks, TODOs, or recommended next steps.

Include specific file paths, command names, test names, branch names, commit hashes, issue IDs, or PR references when they are useful. Do not invent details that are not present in the session context.

## Workflow

1. Identify the project folder name and current local timestamp.
2. Get the current machine hostname.
3. Draft the summary from the session context.
4. Create or update the destination Markdown file in `Daily Notes`.
5. Confirm the final path and mention any missing context or unverified assumptions.
