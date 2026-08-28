---
name: pr-code-review
description: Run bounded, test-backed pull-request code reviews with a persistent findings ledger and a convergent stopping rule. Use when reviewing a PR, branch, commit range, or working-tree diff; validating previously reported review findings; fixing authorized review findings with red-to-green regression tests; answering whether findings are regressions, new branch defects, or incomplete fixes; or preventing repeated review rounds from continually inventing new speculative issues.
---

# PR Code Review

Review an exact diff, require executable evidence for defects, preserve adjudicated
findings in a ledger, and stop when the review has genuinely converged.

## Establish the review contract

Before reviewing:

1. Read repository instructions such as `AGENTS.md` and the relevant build/test
   configuration.
2. Resolve an exact diff range. Prefer the user-supplied range. For a PR or branch,
   use its base branch or merge base through `HEAD`; include the working tree only
   when the user placed it in scope. Never substitute “the codebase.”
3. Identify fixed invariants and specifications from user instructions, repository
   docs, and touched-area contracts. Read only the material relevant to the diff.
4. Derive required test gates from repository instructions and touched workspaces.
   Cross-cutting changes require the repository’s cross-cutting gates.
5. Choose a persistent ledger. Use a user-supplied path, otherwise
   `docs/reviews/<sanitized-branch>-findings.md`. Do not change `.gitignore` merely
   to accommodate the ledger.
6. Record the exact range, base commit, reviewed `HEAD`, and whether working-tree
   changes are included. Snapshot pre-existing worktree changes and preserve them.

If the range, base, or required invariants cannot be determined safely, ask for the
missing information instead of widening scope.

## Choose the round

- Use a first-pass round when no ledger exists for the exact review scope.
- Use a subsequent round when the ledger exists and the scope is unchanged.
- If the ledger already records zero OPEN findings and two consecutive clean rounds,
  stop immediately when the reviewed diff is unchanged.
- If commits or in-scope working-tree changes were added after convergence, review
  only that new surface and reset the consecutive-clean count for the changed scope.
  Preserve all prior ledger rows.

## First pass: find only reproducible defects

Review only changed hunks and the functions directly touched by them. A finding must
be either:

- a concrete violation of a fixed invariant or specification; or
- a correctness, security, performance, or maintainability defect introduced by
  the diff that the author would likely fix.

Do not report style preferences, general subsystem weaknesses, or unrelated defects.

For every candidate finding:

1. Create a focused unit or integration test that expresses the required behavior.
2. Run it and confirm it fails because of the candidate defect, not setup failure.
3. Confirm causality from the diff. Where practical, show that the test passes against
   the base in an isolated worktree; otherwise trace the failure to a changed hunk.
4. Record the test name, exact command, observed failure, and affected location.

When the request is review-only, do not leave temporary PoC edits in the user’s
worktree. Reverse only the exact lines created for the PoC after capturing evidence;
never discard pre-existing user changes. If safe temporary execution is impossible,
record the candidate as an unproven hypothesis.

No failing reproduction means no actionable finding. Put such candidates under
“Unproven hypotheses” with the reason they could not be proved. They are non-blocking.
Do not fix production code during the first pass unless the user separately authorizes
implementation.

## Subsequent rounds: verify, then close

Read the complete ledger before acting. Do not silently re-raise FIXED,
FALSE-POSITIVE, or WONTFIX rows.

### Refute OPEN findings first

For each OPEN row, try to disprove reachability or impact:

- Write or run the test that would fail if the issue were real.
- Trace guards and call paths when a runtime test is not the right proof.
- Mark the row FALSE-POSITIVE only with concrete passing-test or reachability evidence.
- Otherwise keep it OPEN.

### Fix survivors only when authorized

A plain request to review does not authorize production fixes. If remediation is not
authorized, report surviving OPEN rows and stop the implementation portion.

When fixes are authorized, process each surviving OPEN row independently:

1. Add or confirm the focused failing test first and capture the red result.
2. Make the smallest fix that restores the intended invariant.
3. Invert or retain the PoC as a permanent regression test.
4. Run the focused test, then every required test gate.
5. Mark FIXED only after both the focused test and full gates pass.

Use WONTFIX only for an explicit design tradeoff and record the rationale.

### Bound the new-surface scan

After fixing findings, scan only lines changed by this round’s fixes. Apply the same
failing-test evidence bar. Append any reproducible defect with a new ID. Do not use
the fix as permission to rescan the original diff or untouched code.

## Keep the ledger canonical

Never delete existing rows. Use stable IDs and these statuses:

- `OPEN`: blocking and backed by a failing reproduction.
- `FIXED`: permanent regression test and required gates pass.
- `FALSE-POSITIVE`: refuted with concrete evidence.
- `WONTFIX`: explicit accepted tradeoff with rationale.

Use this structure:

```markdown
# Review Ledger — <branch> (<diff range>)

Base: `<sha>`
Reviewed tip: `<sha>`
Working tree included: yes/no
Statuses: OPEN · FIXED · FALSE-POSITIVE · WONTFIX
Blocking findings = OPEN rows with a failing repro test.
Ship when OPEN = 0 and two consecutive rounds are CLEAN.

| ID | Title | Severity | Location | Status | Repro test (name + cmd) | Notes / resolution |
|----|-------|----------|----------|--------|-------------------------|--------------------|
| F-01 | ... | P1/P2/P3 | file:line | OPEN | test_x / `<cmd>` | observed failure |

## Unproven hypotheses (non-blocking)

- H-01: <claim> — could not reproduce because <reason>.

## Round log

- Round 1: <counts and result>.
```

Use P1 for release-blocking or severe security/correctness defects, P2 for material
defects that should be fixed, and P3 sparingly for lower-impact actionable defects.

## Apply the stopping rule

A round is CLEAN when it adds zero new reproducible OPEN findings. Report after every
round:

- status transitions (`OPEN → FIXED/FALSE-POSITIVE/WONTFIX`);
- new reproducible finding count;
- current OPEN count;
- required test-gate results;
- consecutive clean-round count; and
- whether the stopping rule is met.

Stop only when both conditions hold:

1. current OPEN count is zero; and
2. two consecutive rounds are CLEAN against green required gates.

Do not continue reviewing unchanged scope after STOP. Perfect-zero speculation is not
the target; reproducible closure is.

## Return review feedback

Follow any user-provided review-output format. When inline review directives are
available, attach each actionable finding to the shortest useful changed-line range.
Emit no inline directives when there are no actionable findings.

Keep the visible response concise. State plainly when the bounded diff contains zero
reproducible defects.

## Answer provenance and priority challenges

When the user challenges whether findings are regressions, why they were not found
earlier, or whether their priority is correct:

1. Re-read the finding evidence, ledger row, and relevant diff history before
   answering. Do not rely on the current label alone.
2. Classify each challenged finding as one of: true regression, new branch defect,
   incomplete fix, or pre-existing/out-of-scope issue. Do not call every valid
   branch finding a regression.
3. Identify the introduced commit, fix commit, or remaining unfixed commit when the
   evidence supports it. If provenance is uncertain, say what is known and what is
   not proven.
4. Reassess priority from current reachability and impact evidence. Correct P1/P2/P3
   labels when the evidence changes likelihood, impact, or trust-boundary exposure.
5. Answer with a compact table covering finding, classification, introduced/fixed
   provenance, and corrected priority, followed by a brief explanation of why the
   finding was missed or reclassified.
