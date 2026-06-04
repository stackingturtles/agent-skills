---
name: python-uv-pytest-style
description: Use when writing, refactoring, reviewing, or testing Python code, especially services, scripts, FastAPI/data-server code, demo automation, pytest tests, uv-managed projects, typed Python, Ruff formatting/linting, packaging, config parsing, and command-line tools.
---

# Python uv pytest Style

Use this skill for Python implementation, refactoring, testing, and review. Prefer the repo’s local tooling and patterns, especially `uv`, `pytest`, and Ruff where present.

Primary references:

- PEP 8: https://peps.python.org/pep-0008/
- Python typing best practices: https://typing.python.org/en/latest/reference/best_practices.html
- pytest good integration practices: https://pytest.org/en/stable/explanation/goodpractices.html
- Ruff formatter: https://docs.astral.sh/ruff/formatter/
- Ruff linter: https://docs.astral.sh/ruff/linter/
- uv docs: https://docs.astral.sh/uv/

## First Moves

1. Read `pyproject.toml`, nearby modules, and nearby tests before editing.
2. Use the project’s runner. Prefer `uv run ...` when the project uses uv.
3. Add or update a focused pytest test before behavior changes when practical.
4. Keep changes small and local. Avoid broad rewrites or style churn.
5. Run focused tests and format/lint checks available in the project.

Useful commands:

```bash
uv run pytest path/to/test_file.py -q
uv run pytest -q
uv run ruff format .
uv run ruff check .
uv run mypy .   # only when configured
```

## Project Structure

- Follow existing package layout. Do not add new top-level package structures unless needed.
- Keep import side effects minimal. Modules should be safe to import in tests.
- Put command-line entry behavior behind `if __name__ == "__main__":`.
- Keep environment/config parsing at process boundaries. Pass typed config into domain code.
- Prefer small cohesive modules over large mixed-purpose scripts.

## Style

- Format with Ruff when configured; otherwise follow local formatting.
- Use clear names. Avoid clever abbreviations in wide scopes.
- Prefer simple expressions and explicit control flow over dense comprehensions when readability suffers.
- Keep functions small enough to test directly.
- Avoid mutable default arguments.
- Avoid broad `except Exception` unless the boundary is intentional and the error is logged or surfaced clearly.
- Avoid `sys.path` manipulation. Fix packaging/import structure instead.

## Typing

- Add type hints at public/module boundaries, service interfaces, config structures, and non-trivial helpers.
- Do not force exhaustive annotations on obvious short local variables.
- Prefer `dataclass`, `TypedDict`, or Pydantic models for structured data depending on the local stack.
- Use `pathlib.Path` for filesystem paths in new code unless existing APIs expect strings.
- Use precise optionals: handle `None` explicitly rather than relying on truthiness where `0`, `""`, or empty collections are meaningful.

## Errors And Logging

- Raise specific exceptions at library/domain layers; convert to process/API responses at boundaries.
- Preserve context when wrapping exceptions.
- Do not swallow exceptions silently.
- Use structured logging where the service already does.
- Avoid logging secrets, bearer tokens, private keys, signatures, full database URLs, or wallet private keys.

## IO, Network, And External Processes

- Use timeouts for network calls.
- Keep retry behavior bounded and visible.
- Validate external JSON/data shape before using it.
- Use `subprocess.run(..., check=True)` for required commands.
- Keep filesystem writes atomic when losing partial output would matter.

## Tests

- Use pytest idioms: plain `assert`, fixtures for setup, `tmp_path` for filesystem tests, `monkeypatch` for env changes.
- Keep fixtures explicit and local until duplication justifies sharing.
- Test behavior, not implementation details.
- Cover malformed input, missing env vars, timeout/error paths, idempotency, and boundary values.
- Avoid tests that depend on real network, wall-clock timing, global environment, or persistent local files unless marked/integration-scoped.
- Use deterministic seeds for random/demo generation.

## FastAPI / Service Guidance

- Keep route handlers thin: parse request, call domain/service function, return typed response.
- Keep validation in request models or clear helper functions.
- Prefer dependency injection for external clients, signers, and persistence handles.
- Keep startup configuration failures explicit and early.

## Done Criteria

- Focused pytest tests pass.
- Ruff format/check or equivalent local style checks pass when configured.
- Type checks pass when configured.
- No secrets are logged or committed.
- Env/config behavior is documented in templates or README when new variables are introduced.
- The final response states tests/checks run and any skipped checks.
