---
name: go-idiomatic-style
description: Use when writing, refactoring, reviewing, or testing Go/Golang code. Applies idiomatic Go style from Effective Go, Go Code Review Comments, Go Test Comments, Go Doc Comments, Google Go Style, and production Go guidance. Trigger for Go services, CLIs, libraries, HTTP handlers, concurrency, database code, error handling, package design, and tests.
---

# Go Idiomatic Style

Use this skill for Go implementation, refactoring, testing, and review. Treat official Go guidance as primary and production style guides as secondary.

Primary references:

- Effective Go: https://go.dev/doc/effective_go
- Go Code Review Comments: https://go.dev/wiki/CodeReviewComments
- Go Test Comments: https://go.dev/wiki/TestComments
- Go Doc Comments: https://go.dev/doc/comment
- Google Go Style Guide: https://google.github.io/styleguide/go/
- Uber Go Style Guide: https://github.com/uber-go/guide

## First Moves

1. Read nearby packages and tests before editing. Follow local package shape over generic patterns.
2. Identify the smallest package-level behavior change and add or update a focused test first when behavior changes.
3. Prefer standard library tools and local helpers before adding dependencies.
4. Run `gofmt` on edited Go files before testing.
5. Run focused `go test` commands first, then broader tests when feasible.

Useful commands:

```bash
gofmt -w path/to/file.go
go test ./path/to/package
go test ./...
go test -run TestName ./path/to/package
```

## Package Design

- Keep packages small and cohesive. Package names should be short, lowercase, and not repeat the repository or directory hierarchy unnecessarily.
- Avoid `util`, `common`, and `helpers` packages unless the repo already uses them and the abstraction is truly shared.
- Design APIs around behavior, not implementation. Export only what callers need.
- Keep interfaces close to consumers. Do not define an interface just because a concrete type exists.
- Accept interfaces and return concrete types when that improves testability and decoupling.
- Do not add Java-style factories, manager objects, or inheritance-shaped abstractions.

## Naming

- Use short names for narrow scopes and descriptive names for wider scopes.
- Use initialisms consistently: `ID`, `URL`, `HTTP`, `JSON`, `RPC`, `SQL`.
- Name errors as `ErrX` when exported sentinel errors are needed.
- Name error types as `XError` only when callers need structured inspection.
- Avoid stutter: `account.Store`, not `account.AccountStore`, unless local convention differs.

## Errors

- Return errors; do not panic for ordinary failures.
- Wrap errors at boundaries with useful operation context: `fmt.Errorf("load config: %w", err)`.
- Avoid logging and returning the same error unless crossing a process/request boundary.
- Use `errors.Is` and `errors.As` for sentinel and typed errors.
- Keep user-facing HTTP/API error messages separate from internal diagnostics when security or clarity matters.
- Treat ignored errors as a deliberate decision. Use `_ =` only when the error is truly irrelevant, such as best-effort cleanup.

## Context And Concurrency

- Pass `context.Context` as the first argument after the receiver for request-scoped or cancellable work.
- Do not store contexts in structs except for very narrow lifecycle owners.
- Respect cancellation in long-running loops and external calls.
- Prefer simple synchronous code. Add goroutines only when there is real concurrency to manage.
- Use channels for coordination and ownership transfer, not as generic queues when a mutex or direct call is clearer.
- Make shared mutable state explicit and protected.
- Avoid goroutine leaks: every goroutine should have a shutdown path.

## Data And SQL

- Prefer structured APIs and typed structs over map/stringly-typed data.
- Keep SQL placeholders and dialect handling localized.
- Use transactions for multi-step state changes that must be atomic.
- Check `Rows.Err()` after iterating.
- Close rows and response bodies with `defer`.
- Keep migrations idempotent and backward-compatible where existing databases may already exist.

## HTTP And Services

- Keep handlers thin. Parse/validate input, call domain logic, encode response.
- Set content type for JSON responses.
- Use explicit method/path registration where supported by the repo’s Go version.
- Keep CORS, auth, logging, and request limits in middleware or narrow wrappers.
- Avoid global mutable config. Parse env/config once at startup and pass dependencies explicitly.

## Tests

- Use Go’s `testing` package by default. Avoid assertion libraries unless the repo already standardizes on one.
- Prefer table tests when they improve coverage without obscuring the failure.
- Use `got` and `want` names.
- Make tests deterministic: no real network, real clocks, random ordering, or shared global state unless intentionally covered.
- Use `t.Helper()` in test helpers.
- Use `t.Cleanup()` for cleanup.
- Test error cases, boundary values, idempotency, and rollback/retry behavior for stateful code.
- Keep integration tests explicit about external requirements.

## Comments And Documentation

- Write doc comments for exported identifiers that callers use.
- Comments should explain why, invariants, ownership, concurrency, units, and non-obvious tradeoffs.
- Avoid comments that restate the code.
- Keep TODOs actionable and owned when possible.

## Done Criteria

- Edited Go files are `gofmt` formatted.
- Focused tests pass.
- Broader tests are run when the change touches shared behavior; skipped broader tests are reported.
- New exported APIs are documented.
- Error paths are handled and useful.
- Concurrency and context lifetimes are clear.
- The final response includes tests run and any remaining risk.
