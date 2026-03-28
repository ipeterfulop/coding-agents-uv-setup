# Python project rules: use uv for all dependency, environment, and execution tasks

## Table of Contents

- [Core Principle](#core-principle)
- [Dependency Management](#dependency-management)
- [Execution Rules](#execution-rules)
- [Project Structure](#project-structure)
- [Testing](#testing)
- [Linting & Formatting](#linting--formatting)
- [Type Checking](#type-checking)
- [Style Conventions](#style-conventions)
- [Pre-commit](#pre-commit)
- [Hard Constraints (Do Not Violate)](#hard-constraints-do-not-violate)
- [Agent Enforcement Summary](#agent-enforcement-summary)
- [Protect against compromised packages](#protect-against-compromised-packages)
- [How Tos](#how-tos)

## Core Principle

This project uses **uv exclusively** for Python environment and package
management.

Agents MUST:
- Use `uv` for all dependency, environment, and execution tasks.
- Use native `uv` workflows such as `uv add`, `uv remove`, `uv sync`,
  `uv lock`, `uv run`, and `uvx`.
- Avoid all alternative tooling.

Agents MUST NOT:
- Use `pip`, `pip-tools`, `poetry`, `conda`, or similar tools.
- Use `uv pip install` as a substitute for `uv add`.
- Manually create or activate virtual environments.
- Modify dependency lists in `pyproject.toml` directly.

------------------------------------------------------------------------

## Dependency Management

Always use `uv` commands:

- Add dependency: `uv add <package>`
- Add dev dependency by default: `uv add --dev <package>`
- Remove dependency: `uv remove <package>`
- Sync environment: `uv sync`
- Update lockfile: `uv lock`

Rule: Never edit dependencies manually in `pyproject.toml`, and never use
`uv pip install` for normal project dependency management.

------------------------------------------------------------------------

## Execution Rules

Use these categories consistently:

- Dependency and environment changes: `uv add`, `uv remove`, `uv sync`,
  `uv lock`
- Project code and project tools: `uv run ...`
- Temporary, non-persistent dependencies for ad hoc commands: `uv run --with
  <package> ...`
- One-off tools that should not become project dependencies: `uvx <tool>`

Correct:
- `uv add httpx`
- `uv sync`
- `uv run python script.py`
- `uv run python -m module`
- `uv run --with pandas python script.py`
- `uv run pytest`
- `uv run ruff check .`
- `uvx pre-commit install`

Incorrect:
- `python ...`
- `pytest ...`
- `ruff ...`
- `pip install ...`
- `uv pip install ...`

------------------------------------------------------------------------

## Project Structure

Initialization:
- App/script: `uv init <name>`
- Package: `uv init --package <name>`

Project metadata:
- Store project metadata only in `pyproject.toml` (PEP 621).

Forbidden files:
- `setup.py`
- `setup.cfg`
- `requirements.txt`

------------------------------------------------------------------------

## Testing

- Framework: `pytest`
- Run: `uv run pytest`
- Location: `tests/`
- Naming:
  - Files: `test_*.py`
  - Functions: `test_*`

------------------------------------------------------------------------

## Linting & Formatting

Tool: `ruff`

- Lint: `uv run ruff check .`
- Fix: `uv run ruff check --fix .`
- Format: `uv run ruff format .`
- Check format: `uv run ruff format --check .`

Configuration:
- Use `[tool.ruff]` in `pyproject.toml`.

------------------------------------------------------------------------

## Type Checking

Preferred:
- `uv run ty check`

Fallback:
- `uv run mypy .`

------------------------------------------------------------------------

## Style Conventions

- Line length: 88
- Quotes: double
- Indentation: spaces
- Imports: sorted by ruff
- Type ignores must be specific:
  - OK: `# type: ignore[attr-defined]`
  - Not OK: `# type: ignore`

------------------------------------------------------------------------

## Pre-commit

Install via `uvx` only:

- `uvx prek install`
- `uvx pre-commit install`

------------------------------------------------------------------------

## Hard Constraints (Do Not Violate)

- No manual virtualenv handling
- No `pip install`
- No `uv pip install` for dependency management
- No `requirements.txt`
- No `setup.py` usage
- No manual dependency edits

------------------------------------------------------------------------

## Agent Enforcement Summary

If a task involves Python:
1. Use `uv`.
2. Modify dependencies via `uv add`, `uv remove`, `uv sync`, and `uv lock`.
3. Use `uv add --dev` as the default way to add dev dependencies.
4. Run project code and project tools via `uv run`.
5. Use `uv run --with ...` for temporary, non-persistent dependencies.
6. Use `uvx` only for one-off tools.
7. Never bypass uv or fall back to `uv pip install`.

Failure to follow these rules is considered incorrect behavior.

------------------------------------------------------------------------

## Protect against compromised packages


The agent MUST:
- Prefer a project-local `exclude-newer` setting in `pyproject.toml`
  when creating a project, scaffolding `pyproject.toml`, or updating an
  existing `[tool.uv]` table.
- Add or merge the setting below unless the user explicitly opts out or
  asks for a different interval.
- Preserve any existing `[tool.uv]` settings when adding this guardrail.

Agent SHOULD:
- Explain that this reduces the chance of installing a newly published
  malicious package before the community has time to detect and yank it.
- Mention that pip-centered workflows rely more heavily on the wider
  community to discover and report bad uploads.

Preferred default: configure this in the project-local `pyproject.toml`:

```toml
[tool.uv]
exclude-newer = "14 days"
```

Only use a machine-wide default if the user explicitly wants global uv
configuration. In that case, write:

```toml
# ~/.config/uv/uv.toml
exclude-newer = "14 days"
```

Use `"14 days"` as the default value. Change the interval only if the
user explicitly asks for a different cooling-off period.

------------------------------------------------------------------------

## How Tos

Use these workflow rules when the task needs more detail than the core
policy above.

### uv workflows

- Prefer `uv run` for project code and tools already declared in the
  project.
- Prefer `uv run --with <package> ...` when the task needs a temporary
  dependency that should not be recorded in `pyproject.toml`.
- Prefer `uvx` for one-off tools that should not become dependencies.
- Use `uv add --dev <package>` as the default for development-only
  dependencies.
- If the repository already uses dependency groups, ask the user whether
  the existing group scheme should be preserved before adding or changing
  groups.
- Use `uv add --group <name> <package>` only when the repository already
  uses groups or the user explicitly asks for them.
- Use optional dependencies only for user-facing extras that are meant to
  be published.
- If Python version conflicts appear, check both `requires-python` in
  `pyproject.toml` and `.python-version`, then align them with
  `uv python pin`.
- If uv reports `No project table found`, add a minimal `[project]`
  section instead of working around the error with non-uv tooling.

### testing and linting

- Add test tooling with `uv add --dev pytest` when needed.
- Run tests with `uv run pytest`.
- Add Ruff with `uv add --dev ruff` when the project does not already have
  it.
- Use `uv run --with pytest pytest` or similar only for temporary,
  non-persistent test runs.
- Run lint checks with `uv run ruff check .`.
- Auto-fix safe Ruff issues with `uv run ruff check --fix .`.
- For one-off import sorting, prefer `uvx ruff check --select I --fix .`.

### type checking

- Prefer `uv run ty check` when `ty` is already installed in the project.
- Use `uvx ty check .` for one-off runs without adding `ty`.
- If migration from `mypy` to `ty` comes up, note that `ty` is fast but
  does not support mypy plugins such as Pydantic, Django, or SQLAlchemy
  plugins.
- If import resolution is wrong, configure `ty` against the active
  project environment rather than bypassing uv.

### python coding standards

- Prefer LBYL over EAFP: do not use exceptions for normal control flow.
- Use `pathlib.Path` instead of `os.path`.
- Check `.exists()` before `.resolve()` or related path operations.
- Always specify `encoding="utf-8"` for text file reads and writes.
- Keep imports at module scope and prefer absolute imports.
- Keep properties and magic methods O(1).

### legacy tools

- The source material includes `pipenv` references for maintaining legacy
  projects.
- Do not introduce `pipenv` into this repository unless the task is
  explicitly about migration or legacy maintenance.
