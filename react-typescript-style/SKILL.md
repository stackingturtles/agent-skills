---
name: react-typescript-style
description: Use when writing, refactoring, reviewing, or testing React, Next.js, and TypeScript UI code. Applies React official rules, hooks lint guidance, TypeScript component conventions, accessibility, state/effect discipline, frontend testing, and coordinates with frontend-design and vercel-react-best-practices when UI quality or performance matters.
---

# React TypeScript Style

Use this skill for React, Next.js, and TypeScript UI implementation, refactoring, testing, and review. When visual design quality is part of the task, also use the project’s `frontend-design` skill. When performance is central, also use `vercel-react-best-practices`.

Primary references:

- Rules of React: https://react.dev/reference/rules
- React Hooks reference: https://react.dev/reference/react/hooks
- eslint-plugin-react-hooks: https://react.dev/reference/eslint-plugin-react-hooks
- React TypeScript guide: https://react.dev/learn/typescript
- Next.js docs: https://nextjs.org/docs

## First Moves

1. Read nearby components, route files, CSS modules, and existing test patterns.
2. Identify whether the component is server-side, client-side, or mixed. Add `"use client"` only when needed.
3. Keep edits scoped to the requested user-facing behavior.
4. Preserve the existing design system, CSS module patterns, and API helper conventions.
5. Run lint/build or focused tests after changes.

Useful commands:

```bash
npm run lint
npm run build
npm test
```

## Component Design

- Keep components pure. Rendering should not mutate external state, issue network requests, or trigger wallet/browser actions.
- Pass data through props with typed interfaces.
- Derive display values during render when possible instead of storing redundant state.
- Split components when it reduces real complexity, not just to move code around.
- Keep event handlers explicit and narrow.
- Avoid deeply nested JSX. Extract small components when names make behavior clearer.
- Avoid component props that accept many unrelated booleans; prefer a small variant union when needed.

## TypeScript

- Type API responses, config objects, and component props.
- Use discriminated unions for state machines and async statuses when it improves correctness.
- Avoid `any`. Use `unknown` at unsafe boundaries and narrow explicitly.
- Keep parsing/normalization close to data boundaries.
- Use `readonly` arrays/objects where mutation would be surprising.
- Prefer `type` or `interface` according to local convention; do not churn existing style.

## State And Effects

- Use state for values that change over time and affect rendering.
- Do not use effects to compute values that can be derived from props/state in render.
- Keep effects synchronized with external systems only: network polling, browser APIs, subscriptions, wallet providers, timers, storage.
- Clean up subscriptions, intervals, event listeners, and in-flight async work where needed.
- Keep hook dependency arrays correct. Do not suppress hook lint rules without a narrow documented reason.
- Use functional state updates when next state depends on previous state.

## Data Fetching And API Boundaries

- Keep fetch helpers typed and reusable when multiple components need the same endpoint.
- Surface loading, empty, and error states intentionally.
- Do not let stale async responses overwrite newer state.
- Normalize raw protocol units at boundaries and keep formatting helpers tested for financial values.
- Keep secrets and server-only values out of client bundles.

## Forms And Actions

- Validate user input before enabling actions.
- Keep wallet/transaction flows explicit: preview, request signature/transaction, wait for receipt, refresh state, show user-readable errors.
- Disable buttons only for clear reasons and show actionable status text.
- Avoid hidden long-running states that block unrelated UI.
- Use accessible labels for inputs and controls.

## Accessibility

- Use semantic HTML first.
- Buttons should be buttons; navigation should be links.
- Use labels for form inputs.
- Preserve keyboard access and visible focus states.
- Use `aria-*` only when native semantics are insufficient.
- Ensure status/error messages are visible and understandable.

## Styling

- Follow local CSS module and styleguide conventions.
- Keep layout stable with explicit responsive constraints for grids, tables, panels, and controls.
- Avoid text overflow and overlap at mobile and desktop widths.
- Prefer system/design tokens already present in the repo.
- Do not introduce decorative UI patterns that conflict with the existing product style.

## Tests And Verification

- Test pure formatters, parsing helpers, and state reducers with fast unit tests.
- Use component/browser tests for interactions that are hard to validate by inspection.
- For significant UI layout changes, verify desktop and mobile rendering.
- Run `npm run lint` and `npm run build` when feasible.
- If a local dev server is relevant and available, use browser verification for key flows.

## Done Criteria

- Types are explicit at API and component boundaries.
- Hooks follow React rules and dependency correctness.
- UI has loading, empty, error, disabled, and success states where applicable.
- Inputs and actions are accessible.
- Layout does not overlap or overflow at expected breakpoints.
- Focused tests, lint, and build are run when feasible; skipped checks are reported.
