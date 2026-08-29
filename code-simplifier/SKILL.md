---
name: code-simplifier
description: Simplify recently modified code for clarity, consistency, and maintainability while preserving behavior. Use when the user asks to simplify, clean up, clarify, or reduce complexity in existing code without changing functionality.
---

# Code Simplifier

Simplify scoped code without changing its externally observable behavior. Prefer
readable, explicit code over compact or clever alternatives.

## Establish the scope

1. Read the repository instructions and relevant build, formatting, linting, and
   test configuration before editing.
2. Use the scope supplied by the user. Otherwise, focus on code modified in the
   current session or working-tree diff and the functions directly affected by it.
3. Inspect pre-existing worktree changes and preserve unrelated user work.
4. Identify the behavior and test gates that must remain unchanged.

Do not turn a focused simplification request into a broader refactor or cleanup.

## Simplify for clarity

Apply the repository's language and project conventions. Look for changes that
materially improve readability or maintainability, such as:

- reducing unnecessary nesting and control-flow complexity;
- removing genuine duplication, dead indirection, or redundant abstractions;
- improving unclear names and grouping related logic;
- replacing nested ternaries and dense one-liners with explicit control flow;
- removing comments that only restate obvious code while preserving comments that
  explain intent, invariants, constraints, or non-obvious tradeoffs; and
- making error handling and state transitions easier to follow without changing
  their semantics.

Choose clarity over fewer lines. Preserve abstractions that establish useful
boundaries, express domain concepts, support testing, or make future changes safer.
Do not impose language-specific preferences that are absent from the repository's
instructions and surrounding code.

## Preserve behavior

- Keep public APIs, outputs, side effects, ordering, error behavior, and supported
  inputs unchanged unless the user explicitly authorizes a behavioral change.
- Avoid speculative performance rewrites or dependency changes.
- Prefer the smallest coherent simplification when several approaches are possible.
- Leave code unchanged when a proposed rewrite is merely stylistic or makes the
  behavior harder to verify.

## Verify the result

Review the final diff for accidental scope expansion. Run the most focused relevant
tests first, followed by the repository's required formatting, linting, type-checking,
and test gates in proportion to the change. If a required check cannot run, state
what was not verified and why.

Report only the significant simplifications and the verification performed. State
plainly when the scoped code does not have a worthwhile behavior-preserving
simplification.
