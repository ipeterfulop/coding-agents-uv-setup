# Python Project Rules

## Table of Contents

- [Toolchain](#toolchain)
- [Package Operations](#package-operations)
- [Running Python and Tools](#running-python-and-tools)
- [Project Initialisation](#project-initialisation)
- [Testing](#testing)
- [Linting and Formatting](#linting-and-formatting)
- [Type Checking](#type-checking)
- [Code Style Conventions](#code-style-conventions)
- [Pre-commit Hooks](#pre-commit-hooks)
- [Hard Constraints](#hard-constraints)
- [How to-s](#how-to-s)
  - [`uv run` vs `uvx`](#uv-run-vs-uvx)
  - [Python versions](#python-versions)
  - [Virtual environment location](#virtual-environment-location)
  - [Migrating from `requirements.txt`](#migrating-from-requirementstxt)
  - [Optional dependencies vs dependency groups](#optional-dependencies-vs-dependency-groups)
  - [Troubleshooting](#troubleshooting)
  - [Jupyter](#jupyter)
  - [Tests with pytest](#tests-with-pytest)
  - [Ruff linting and import sorting](#ruff-linting-and-import-sorting)
  - [ty type checker](#ty-type-checker)

## Toolchain

This project uses **uv** as its sole package and environment manager.
Never use `pip`, `pip-tools`, `poetry`, `conda`, or any other package manager.
Never activate or create virtual environments manually — uv owns `.venv/` entirely.

## Package Operations

| Intent | Command |
|---|---|
| Add a runtime dependency | `uv add <package>` |
| Add a dev-only dependency | `uv add --dev <package>` |
| Remove a dependency | `uv remove <package>` |
| Sync the environment to the lockfile | `uv sync` |
| Regenerate the lockfile | `uv lock` |

> Always modify dependencies through `uv add` / `uv remove`. Never edit `pyproject.toml` dependency lists by hand.

## Running Python and Tools

Always prefix execution with `uv run`. Direct invocations of `python`, `pytest`, `ruff`, or any other tool are forbidden — they may resolve to the wrong environment or a system binary.

```bash
uv run python script.py          # run a script
uv run python -m module_name     # run a module
uv run pytest                    # run tests
uv run ruff check .              # lint
uvx <tool>                       # one-off tool that is not a project dependency
```

## Project Initialisation

- General application or script: `uv init project-name`
- Distributable library or package: `uv init --package project-name`

Project metadata lives exclusively in `pyproject.toml` (PEP 621).
Never create `setup.py`, `setup.cfg`, or `requirements.txt`.

## Testing

- Framework: **pytest**
- Entry point: `uv run pytest`
- Test root: `tests/` at the project root
- File naming convention: `test_*.py`
- Function naming convention: `test_*`
- No `__init__.py` required inside `tests/`

## Linting and Formatting

Tool: **ruff** (covers both linting and formatting).

```bash
uv run ruff check .              # lint
uv run ruff check --fix .        # lint and apply safe auto-fixes
uv run ruff format .             # format
uv run ruff format --check .     # verify formatting without writing
```

Configuration lives under `[tool.ruff]` in `pyproject.toml`.

## Type Checking

Preferred tool: **ty** (if configured), fallback: **mypy**.

```bash
uv run ty check      # with ty
uv run mypy .        # with mypy
```

Configuration lives in `pyproject.toml`.

## Code Style Conventions

- Line length: 88 characters (ruff default)
- Quotes: double
- Indentation: spaces
- Import sorting: managed by ruff (`select = ["I"]` — isort ruleset)
- Never add bare `# type: ignore` — always include the specific error code (e.g. `# type: ignore[attr-defined]`)

## Pre-commit Hooks

Preferred tool: **prek**; alternative: **pre-commit**.
Always install via `uvx` — never via pip.

```bash
uvx prek install          # install prek hooks
uvx pre-commit install    # install pre-commit hooks
```

## Hard Constraints

The following actions are always wrong in this project:

- Creating or activating a virtual environment manually
- Installing packages with `pip install` (globally or locally)
- Using `requirements.txt` for dependency management
- Running `python setup.py` in any form
- Adding dependencies directly to `pyproject.toml` by hand instead of via `uv add`

## How to-s

Reference patterns for **uv** (beyond the rules above). Prefer `uv run` for anything that must use this project's dependencies; use `uvx` (alias for `uv tool run`) for standalone tools in an isolated environment.

### `uv run` vs `uvx`

| Use | Command | When |
|-----|---------|------|
| Project code, tests, apps | `uv run ...` | Needs packages from `pyproject.toml` / lockfile |
| One-off or global-style tools | `uvx ...` | Tool should not depend on the project env |

```bash
# uv run — uses project dependencies
uv run python script.py
uv run pytest tests/
uv run flask run

# uvx — isolated environment, independent of the project
uvx ruff format .
uvx cookiecutter gh:user/template
uvx django-admin startproject mysite
```

### Python versions

| Goal | Command / Setting |
|------|-------------------|
| Declare supported range | `requires-python = ">=3.11"` in `[project]` of `pyproject.toml` |
| Pin a single version for the tree | `uv python pin 3.12` — writes/updates `.python-version`; must satisfy `requires-python` |
| Override for one command only | `uv run --python 3.10 <cmd>` — no file changes |
| List available versions | `uv python list` |
| Install a specific version | `uv python install 3.12` |

**What is `.python-version`?** A single-line file containing the exact version (e.g. `3.11.5`) that uv (and pyenv) use to determine the Python interpreter for this project. It controls the developer's machine; `requires-python` in `pyproject.toml` controls package compatibility for end-users. The pinned version must satisfy `requires-python`.

### Virtual environment location

Default env is `.venv/` at the project root.

| Method | Scope | How |
|--------|-------|-----|
| `--active` flag | Single command | Activate a venv first, then `uv sync --active` / `uv add --active` |
| `UV_PROJECT_ENVIRONMENT=path` in shell config | System-wide | Add `export UV_PROJECT_ENVIRONMENT=".env"` to `~/.zshrc` |
| `UV_PROJECT_ENVIRONMENT=path <cmd>` | Single command | `UV_PROJECT_ENVIRONMENT=custom-venv uv sync` |
| `.envrc` with direnv | Per-project | `export UV_PROJECT_ENVIRONMENT=custom-venv` in `.envrc` |

### Migrating from `requirements.txt`

```bash
uv init --bare                          # create minimal pyproject.toml
uv add -r requirements.txt              # import runtime deps
uv add --dev -r requirements-dev.txt    # import dev deps (if separate)
uv pip freeze                           # verify
rm requirements.txt requirements-dev.txt
```

Do not use `requirements.txt` for ongoing dependency management.

### Optional dependencies vs dependency groups

| Feature | Optional dependencies | Dependency groups |
|---------|-----------------------|-------------------|
| Config section | `[project.optional-dependencies]` | `[dependency-groups]` |
| Published to PyPI | Yes | No |
| Install syntax | `pip install package[extra]` | `uv sync --group <name>` |
| Primary use | End-user feature sets | Dev tools (pytest, ruff, mypy) |

```bash
uv add --optional aws boto3 s3fs          # user-facing optional extra
uv add --group test pytest pytest-cov     # dev-only dependency group
uv add --group lint ruff mypy
uv sync --group test                      # install a specific group
uv sync --all-groups                      # install all groups
```

### Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `No project table found` | `pyproject.toml` has no `[project]` section | Add `[project]` with `name` and `version`, or use `--no-project` flag |
| `.python-version` vs `requires-python` mismatch | Pinned version doesn't satisfy the project requirement | `uv python pin <compatible-version>`, adjust `requires-python`, or use `uv run --python` temporarily |
| `ModuleNotFoundError` during install/build | Build isolation excludes deps not in `[build-system].requires` | Add missing deps to `[build-system].requires`; quick workaround: `uv pip install --no-build-isolation` |

### Jupyter

```bash
# Ad hoc (temporary environment)
uv run --with jupyter --with pandas --with matplotlib jupyter lab

# Persistent (project dependency)
uv add --dev jupyter
uv run jupyter lab
```

### Tests with pytest

```bash
uv run pytest                             # pytest is a project dependency
uv run --with pytest pytest               # run without adding as dependency
uv run pytest tests/test_api.py           # specific file
uv run pytest tests/test_api.py::test_fn  # specific function
uv run pytest -v -x -s                    # verbose, stop on first fail, show prints
uv run pytest --cov=mypackage --cov-report=term-missing  # with coverage
```

Add as a dev dependency for real projects:

```bash
uv add --group test pytest pytest-cov
```

Configure in `pyproject.toml`:

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --tb=short"
```

### Ruff linting and import sorting

```bash
uv run ruff check .               # lint
uv run ruff check --fix .         # lint and auto-fix
uv run ruff format .              # format
uv run ruff format --check .      # verify formatting without writing
uvx ruff check --select I --fix . # one-off import sort without installing
```

Recommended rule set in `pyproject.toml`:

```toml
[tool.ruff.lint]
extend-select = [
    "F",        # Pyflakes — undefined vars, unused imports
    "E", "W",   # PyCodeStyle — PEP 8
    "I",        # isort — import ordering
    "UP",       # PyUpgrade — modernise syntax
    "C4",       # Comprehensions
    "FA",       # Future annotations
    "ISC",      # Implicit string concat
    "RET",      # Return practices
    "SIM",      # Simplifications
    "PTH",      # Pathlib over os.path
    "TC",       # Type-checking block
]
```

### ty type checker

```bash
uvx ty check .                        # run without installing
uv add --dev ty && uv run ty check .  # add to project
uv tool install ty@latest             # install globally
ty check . --output-format concise    # concise output
ty check . --python .venv             # specify environment
```

Update:

```bash
uv lock --upgrade-package ty   # update project dependency
uv tool upgrade ty             # update global installation
```
