# Modern Python Checklists

## Recommended Tooling

| Area | Tool | Purpose |
| --- | --- | --- |
| Python version | Python 3.12+ | Modern typing and syntax |
| Project manager | uv | Virtualenv, dependency sync, lock files, command execution |
| Config | pyproject.toml | Project metadata and tool configuration |
| Formatter | Ruff formatter | Deterministic code layout |
| Linter | Ruff | Static hygiene, import sorting, bug-prone patterns |
| Type checker | mypy | Interface and type consistency |
| Tests | pytest | Behavior verification |
| Coverage | pytest-cov | Line and branch coverage signal |
| Local hooks | pre-commit | Submit-time local checks |
| CI | GitHub Actions | Clean-environment verification |

## Common Commands

Prefer `uv run ...` when the project uses uv:

```bash
uv sync --locked --dev
uv run ruff format .
uv run ruff format --check .
uv run ruff check .
uv run ruff check . --fix
uv run mypy src
uv run pytest
uv run pytest --cov=src --cov-report=term-missing
uv run pre-commit run
```

If using a local `.venv` directly on Windows:

```powershell
.\.venv\Scripts\ruff.exe format --check .
.\.venv\Scripts\ruff.exe check .
.\.venv\Scripts\mypy.exe src
.\.venv\Scripts\pytest.exe --cov=src --cov-report=term-missing
```

## pyproject.toml Baseline

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
pythonpath = ["src"]
addopts = [
    "-ra",
    "--strict-markers",
    "--strict-config",
]

[tool.ruff]
line-length = 88
target-version = "py313"

[tool.ruff.lint]
select = [
    "E",
    "F",
    "I",
    "B",
    "UP",
    "SIM",
]

[tool.mypy]
python_version = "3.13"
strict = true
warn_return_any = true
warn_unused_configs = true

[tool.coverage.run]
branch = true
source = ["src"]

[tool.coverage.report]
show_missing = true
skip_covered = true
```

Adjust `target-version` and `python_version` to the repository's configured Python version.

## pre-commit Baseline

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v6.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-toml
      - id: check-added-large-files
      - id: check-merge-conflict

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.15.12
    hooks:
      - id: ruff-check
      - id: ruff-format
```

Install and run:

```bash
uv run pre-commit install
uv run pre-commit run --all-files
```

After installation, normal commits automatically run pre-commit for the committed files. Use `uv run pre-commit run` for a quick manual check; reserve `--all-files` for setup, hook config changes, or broad cleanup.

## GitHub Actions Baseline

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

permissions:
  contents: read

jobs:
  quality:
    name: Quality
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v6

      - name: Install uv
        uses: astral-sh/setup-uv@v7
        with:
          enable-cache: true
          cache-dependency-glob: uv.lock

      - name: Sync dependencies
        run: uv sync --locked --dev

      - name: Check formatting
        run: uv run --locked ruff format --check .

      - name: Lint
        run: uv run --locked ruff check .

      - name: Type check
        run: uv run --locked mypy src

      - name: Test
        run: uv run --locked pytest --cov=src --cov-report=term-missing
```

## Final Verification Checklist

- Project uses a clear source layout, preferably `src/`.
- `pyproject.toml` contains project metadata and tool config.
- Runtime dependencies and development dependencies are separated.
- Lock file is committed when the project uses a lockfile-based manager.
- Public functions and methods have precise type annotations.
- Ruff formatting and linting pass.
- mypy passes without broad ignores.
- pytest covers normal, error, and boundary behavior.
- Coverage report is reviewed for meaningful missing branches.
- pre-commit is configured for fast local checks.
- CI runs check-only formatter, lint, type check, and tests in a clean environment.
- Git commits are focused and use clear Conventional Commit messages.
