---
name: read-context
description: "Read core repository context before working: README.md, CLAUDE.md, AGENTS.md, docs/*, and git-tracked files. Use when the user asks to read context, understand a codebase, onboard to a repository, or inspect project structure before making changes."
---

Read repository context to understand the codebase before answering or making changes.

When this skill is invoked:

1. Read `README.md` if it exists.
2. Read `CLAUDE.md` if it exists.
3. Read `AGENTS.md` if it exists.
4. Read files under `docs/*` if the `docs/` directory exists.
   - Prefer reading relevant markdown/text files directly.
   - If `docs/` contains many files, list them first and prioritize overview, architecture, setup, requirements, and planning documents.
5. Run:

```bash
git ls-files
```

6. Use the collected context to summarize the codebase structure, key technologies, conventions, and likely next steps.

Do not modify files while performing this context read unless the user separately asks for changes.
