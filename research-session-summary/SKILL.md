---
name: research-session-summary
description: Summarize research sessions into archival Markdown notes in an Obsidian vault. Use when the user asks to capture, archive, summarize, hand off, or preserve research, investigation, exploration, reading, synthesis, conclusions, teachings, evidence, notes, or next research steps.
---

# Research Session Summary

Create a factual Markdown summary of a research session and store it in the user's Obsidian vault for later retrieval.

## Output Location

Save the completed note under:

```text
~/current-obsidian-vault/Daily Notes
```

Use this filename format:

```text
<project-folder-name>-research-YYMMDDHHMM.md
```

Derive `<project-folder-name>` from the current workspace/repository directory name. Use the local current time for `YYMMDDHHMM`. If the workspace is not inside a project folder, use the most specific meaningful directory name available.

## Frontmatter

Do not use an Obsidian template. Start the note with this filled-out YAML frontmatter:

```markdown
---
project: <project-folder-name>
created: YYYY-MM-DD HH:MM
type: research
hostname: <hostname>
---
```

Use the same project folder name used in the filename for `project`. Use local time for `created`. Get `hostname` from the current PC/Mac, for example with `hostname` on macOS/Linux. Keep `type` exactly `research`.

## Summary Requirements

Summarize the research session using the available conversation history, source material, notes, terminal output, documents created or edited, analysis artifacts, plans, decisions, and any relevant context from the session.

The summary should be factual, concise, and useful for future readers. Prioritize conclusions, teachings, evidence, unresolved questions, and next exploration steps over a transcript of activity. Do not include generic commentary or speculation. If something is uncertain, say so clearly.

Structure the document as follows after the frontmatter:

```markdown
# Research Session Summary

## Highlights

- Identified the strongest evidence for the preferred approach.
- Clarified the main tradeoff between options A and B.
- Created detailed notes under `docs/research/...`.
- Left open questions around implementation cost and operational risk.

## Conclusions And Teachings

Write concise paragraphs explaining the most important conclusions, what the reader should learn, and why those conclusions matter.

## Visual Summary

Use Markdown tables where they clarify comparisons, decisions, evidence, source quality, timelines, or tradeoffs.

Use Mermaid diagrams when they make relationships, flows, decision trees, causal links, or research maps easier to understand.

## Detailed Notes Created

List detailed notes created or materially updated during the session. Include each path relative to the project root.

## Next Steps For Exploration

List focused next research steps, experiments, questions to answer, sources to consult, or decisions to make.

## Detailed Summary

Write a series of coherent paragraphs explaining the session in chronological or logical order.
```

Highlights should be a short bulleted list of the most important outcomes from the session. Include only high-signal items, such as conclusions reached, important teachings, key evidence, created notes, decisions, resolved ambiguities, and remaining risks.

The visual summary should be included when it adds clarity. Prefer a compact table for comparisons, evidence matrices, option tradeoffs, source summaries, open questions, or next-step prioritization. Prefer a Mermaid diagram for relationships, workflows, decision trees, timelines, or dependencies. Do not force a graph if prose or a table is clearer.

The detailed notes section must list any detailed notes created or materially updated during the research session. Use project-root-relative paths such as `docs/research/model-selection.md`. If no detailed notes were created, write `No separate detailed notes were created during this session.`

The detailed summary should read like a research handoff note, not a transcript. Group related work together, avoid excessive detail, and preserve enough context that someone can understand the research later without rereading the full session.

Cover the following where relevant:

- The original research question, goal, or decision being supported.
- Sources, documents, repositories, tools, or datasets consulted.
- Important findings and the evidence behind them.
- Conclusions and teachings the reader should carry forward.
- Tradeoffs, alternatives compared, and why some were rejected.
- Visual synthesis through tables or Mermaid diagrams where useful.
- Notes, files, or artifacts created, with paths relative to project root.
- Assumptions, uncertainties, contradictions, or gaps in the evidence.
- Recommended next steps for exploration, validation, or implementation.

Include specific source names, URLs, file paths, command names, document names, issue IDs, or references when they are useful. Do not invent details that are not present in the session context.

## Workflow

1. Identify the project folder name and current local timestamp.
2. Get the current machine hostname.
3. Determine which detailed notes were created or materially updated, and record paths relative to the project root.
4. Draft the summary from the research session context.
5. Add a visual summary with tables or Mermaid diagrams when it improves comprehension.
6. Create or update the destination Markdown file in `Daily Notes`.
7. Confirm the final path and mention any missing context or unverified assumptions.
