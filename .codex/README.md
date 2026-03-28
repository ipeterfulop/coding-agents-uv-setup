# Make Codex Use `uv`

Codex works better when the repository gives it direct, local instructions.
For Python projects that should use `uv` consistently, this directory already
contains a ready-made [AGENTS.md](/Users/peterfulop.me/code/coding-agents-uv-setup/.codex/AGENTS.md)
that tells Codex how to install dependencies, run tools, and avoid the usual
fallbacks to `pip` or raw `python`.

This guide explains how to use that file as the default instruction set for
Codex sessions.

## Why This Matters

Without a clear project policy, Codex may choose commands that technically work
but do not match the repository standard. In a Python project, that usually
means one of these:

- `pip install` instead of `uv add`
- `python script.py` instead of `uv run python script.py`
- direct tool execution instead of `uv run pytest` or `uv run ruff check .`
- manual edits to dependency declarations

The bundled [AGENTS.md](/Users/peterfulop.me/code/coding-agents-uv-setup/.codex/AGENTS.md)
is meant to prevent that.

## Quick Verification

Before you refine the wording, it helps to know what good behavior looks like.
Open Codex in the repository and try one or two prompts such as:

- "Add `polars` as a dependency"
- "Create and run a Polars script that groups sales by city"

If the configuration is working, Codex should prefer commands like:

- `uv add polars`
- `uv run python <script>.py`

If it reaches for `pip install`, plain `python`, or `uv pip install`, the
repository instructions are not specific enough yet.

## Use the Bundled `AGENTS.md`

Treat the bundled
[AGENTS.md](/Users/peterfulop.me/code/coding-agents-uv-setup/.codex/AGENTS.md)
as the canonical instruction file for Codex.

For a repository that should follow the same rules:

1. Copy this file into the project root as `AGENTS.md`.
2. Start Codex from that repository.
3. Let the project-level `AGENTS.md` guide dependency changes, execution, and
   tool usage.

If you already have an `AGENTS.md`, compare it against the bundled version and
keep one clear source of truth. Mixing overlapping rule sets usually makes the
agent less predictable.

## What the Included File Covers

The bundled [AGENTS.md](/Users/peterfulop.me/code/coding-agents-uv-setup/.codex/AGENTS.md)
already gives Codex the important operational rules:

- use `uv` exclusively for Python package and environment management
- change dependencies with `uv add`, `uv remove`, `uv sync`, and `uv lock`
- add dev dependencies with `uv add --dev` by default
- run project code and developer tools through `uv run`
- use `uv run --with ...` for temporary, non-persistent dependencies
- use `uvx` only for one-off tools
- use `pytest`, `ruff`, and `ty` or `mypy` through `uv`
- ask before preserving or changing an existing dependency-group scheme
- avoid `pip`, `uv pip install`, manual virtualenv activation, and manual
  dependency edits

It also includes extra guidance for linting, testing, type checking, coding
style, and common `uv` workflows.

## Common Mistakes

The most common setup mistakes are straightforward:

- keeping the reference `AGENTS.md` in a side directory but not copying it into
  the target repository
- having two different `AGENTS.md` files with overlapping or conflicting rules
- telling Codex to use `uv` without also specifying how to run tests and tools
- allowing exceptions that reintroduce `pip` or manual environment handling

In practice, the cleanest approach is to start from the bundled file and change
as little as possible.

## Example Test With Polars

If you want a concrete smoke test, use a task that forces Codex to both add a
dependency and execute code.

Prompt:

```text
Add polars to this project, create a script that builds a small sales DataFrame,
computes a revenue column, groups by city, sums revenue, and prints the result.
Then run the script. Do not use pip.
```

A suitable script would look like this:

```python
import polars as pl

df = pl.DataFrame(
    {
        "city": ["Budapest", "Budapest", "Vienna", "Prague"],
        "product": ["book", "pen", "book", "pen"],
        "qty": [3, 10, 5, 7],
        "unit_price": [12.5, 1.8, 11.0, 1.5],
    }
).with_columns((pl.col("qty") * pl.col("unit_price")).alias("revenue"))

summary = (
    df.group_by("city")
    .agg(pl.sum("revenue").round(2).alias("revenue"))
    .sort("revenue", descending=True)
)

print(summary)
```

The expected behavior is `uv add polars` followed by `uv run python ...`. If
Codex edits dependency files manually or uses `pip install` or `uv pip install`,
the instructions still need work.

## When To Customize

Copy the bundled file as-is unless the target repository genuinely needs extra
rules. Good reasons to customize it include:

- adding framework-specific commands
- naming project-specific dependency groups
- documenting required test or lint entrypoints
- recording constraints for generated code in that repository

Keep the `uv` rules intact unless the project is intentionally using a
different package-management strategy.
