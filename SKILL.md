---
name: modern-python-development
description: "Use when implementing, reviewing, scaffolding, or maintaining Python projects that should follow modern Python engineering conventions: src layout, pyproject.toml, uv dependency management, Ruff formatting and linting, mypy strict typing, pytest tests, coverage, pre-commit, Git workflow, or GitHub Actions CI."
---

# Modern Python Development

## Overview

Practice modern Python as an engineering workflow, not as isolated style advice. Keep project structure, tooling, types, tests, local checks, Git history, and CI aligned so every change is readable, verifiable, and maintainable.

## New Project Setup

When creating a new Python project, default to `uv` for project management, dependency management, virtualenv handling, lock files, and command execution unless the user or repository explicitly chooses another tool. Establish the quality workflow before adding meaningful features:

1. Choose the package name and Python target version with the user or repository constraints.
2. Initialize project metadata in `pyproject.toml`, pin the Python version, and commit a lock file when the package manager supports it.
3. Create `src/<package_name>/` and `tests/` immediately. Add a tiny importable module and one test so imports, packaging, and pytest discovery are verified from the start.
4. Add development dependencies for Ruff, mypy, pytest, pytest-cov, and pre-commit. Keep runtime dependencies separate from development dependencies.
5. Configure Ruff, mypy, pytest, and coverage in `pyproject.toml`.
6. Add `.gitignore` entries for virtualenvs, caches, coverage data, and build outputs. Do not ignore `pyproject.toml`, lock files, source, tests, docs, or CI config.
7. Add `.pre-commit-config.yaml` for fast local checks.
8. Add CI that runs locked dependency sync, Ruff format check, Ruff lint, mypy, and pytest with coverage.
9. Run the full local verification sequence once before calling the scaffold complete. After that, use normal pre-commit hooks through `git commit` or `pre-commit run` instead of repeating all-files checks constantly.

Load `references/checklists.md` for common commands and baseline configuration snippets.

## Operating Workflow

1. Inspect the repository before changing code: read `pyproject.toml`, `.python-version`, dependency lock files, `src/`, `tests/`, `.pre-commit-config.yaml`, and CI workflows if present.
2. Prefer the existing toolchain. If the project uses `uv`, run tools through `uv run`; if it has a checked-in `.venv`, direct executables are acceptable for local verification.
3. Keep source under `src/<package>/` and tests under `tests/`. Do not put importable production modules in the repository root unless the project already intentionally uses that layout.
4. Put shared Python tool configuration in `pyproject.toml` when supported by the tool.
5. Write or update tests with behavior in mind before or alongside implementation. Cover normal paths, error paths, and boundary cases.
6. Run verification in this order: formatter, linter, type checker, tests, and coverage when relevant. After pre-commit is configured, rely on `git commit` to trigger hooks, or run `pre-commit run` manually for a quick local check.
7. If a check fails, diagnose the category first: formatting, lint, typing, behavior, environment, or generated-file permission. Do not paper over failures with `Any`, broad ignores, or skipped tests.

For command and configuration examples, load `references/checklists.md` when you need exact snippets.

## Project Structure

Use this shape for new packages unless the repository already has a stronger convention:

```text
project/
+-- pyproject.toml
+-- README.md
+-- uv.lock
+-- .python-version
+-- src/
|   +-- package_name/
|       +-- __init__.py
|       +-- module.py
+-- tests/
    +-- test_module.py
```

`src/` layout matters because tests then import the package closer to how users and CI will import it. It exposes packaging and import-path mistakes earlier than root-level modules.

## Code Standards

Use Ruff formatter for mechanical layout and Ruff linter for static hygiene. Humans should spend review attention on names, boundaries, behavior, tests, and tradeoffs.

Prefer:

- 4-space indentation, `snake_case` functions and variables, `PascalCase` classes, `UPPER_CASE` constants.
- Sorted imports grouped as standard library, third-party, local package.
- Comments that explain why, not comments that restate obvious code.
- Small functions with honest names and explicit contracts.
- Modern syntax for the configured Python target, such as `list[str]`, `dict[str, int]`, and `str | None`.

Avoid:

- Root-level production modules in package projects.
- Unused imports, unused variables, broad `except Exception`, hidden global state, and clever expressions that reduce readability.
- Manual formatting debates when the formatter can decide.

## Typing Standards

Treat type annotations as interface contracts for humans, IDEs, and static analysis.

- Annotate all public functions and methods with parameter and return types.
- Write `-> None` for functions that perform actions and return no business value.
- Use precise types before reaching for `Any`.
- Keep `Any` at system boundaries such as raw JSON, untyped third-party APIs, or dynamic config; narrow it quickly.
- Express failure honestly: return `T | None` only when callers should handle absence; raise a clear exception when invalid input is an error.
- Do not silence mypy to force a design through. Change the interface, narrow the type, or isolate the untyped boundary.

## Testing Standards

Use pytest to verify behavior. Ruff and mypy can prove style and type consistency; they cannot prove business logic.

Write tests that:

- Name the behavior and expected result, for example `test_divide_rejects_zero`.
- Use direct `assert` statements with meaningful expected values.
- Cover normal paths, invalid inputs, edge values, and branch behavior.
- Use `pytest.raises(..., match=...)` for expected exceptions and error messages.
- Use `@pytest.mark.parametrize` when one behavior needs multiple inputs.
- Use fixtures only when they clarify shared setup; keep simple data inline.
- Use `tmp_path` for file I/O tests rather than writing to the project root or user directories.
- Test external behavior, not implementation details.

Coverage is a signal, not a goal by itself. A test without an assertion may execute code while proving nothing.

## Local Automation

Use pre-commit to move fast checks before commit, but keep CI as the final gate. Good pre-commit candidates are Ruff formatting, Ruff linting, YAML/TOML checks, trailing whitespace, end-of-file fixes, large-file checks, and merge-conflict checks.

After `.pre-commit-config.yaml` is configured and `pre-commit install` has been run, keep daily use lightweight:

- Let `git commit` trigger hooks for the files being committed.
- Run `pre-commit run` when you want a quick manual check of the current staged changes.
- Run `pre-commit run --all-files` mainly after changing hook configuration, when introducing pre-commit to an existing repository, or before a larger PR if you want a full sweep.

If a hook modifies files, inspect the diff and stage the modified files again before committing. Do not rely on `--no-verify` except for a clearly understood emergency with CI as a backstop.

## Git And CI

Use Git history to preserve intent:

- Work on focused branches such as `feat/...`, `fix/...`, `docs/...`, `test/...`, `chore/...`, or `ci/...`.
- Review `git diff` before staging and `git diff --staged` before committing.
- Keep each commit to one clear purpose.
- Use Conventional Commits such as `feat: add gradebook summary helpers`, `test: cover calculator edge cases`, or `ci: add python quality workflow`.
- In PR descriptions, include summary and verification commands.

CI should run in a clean environment on push and pull requests. Prefer locked dependency sync, check-only formatter commands, lint, mypy, and pytest with coverage. CI should reveal drift between `pyproject.toml` and lock files; it should not silently rewrite them.

## Completion Gate

Before claiming a Python change is complete, report the exact verification commands and their outcomes. If a command could not run, state the blocker and whether it appears to be an environment issue or a project issue.
