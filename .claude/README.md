# Make Claude Code Use `uv`


- [Why This Matters](#why-this-matters)
- [Quick Verification](#quick-verification)
- [Use the Bundled `CLAUDE.md`](#use-the-bundled-claudemd)



Claude Code can fall back to `pip` and plain `python` unless the repository
spells out a preferred workflow. If you want package changes, script execution,
and tool invocations to stay inside `uv`, use the bundled `CLAUDE.md` in this
directory as your source of truth.

This guide starts with a quick behavior check, then shows how to reuse the
included `CLAUDE.md` file in a project root or as a broader default for other
projects.

## Why This Matters

When the package manager is ambiguous, Claude Code may choose commands that do
not match the rest of the project. That usually shows up in a few ways:

- dependencies installed with `pip` instead of `uv add`
- scripts run with raw `python` instead of `uv run`
- lockfiles and environments drifting out of sync

Adding a short repository rule set avoids that ambiguity. This repository
already includes one in [CLAUDE.md](/Users/peterfulop.me/code/coding-agents-uv-setup/.claude/CLAUDE.md).

## Quick Verification

Before you refine the wording, it helps to know what good behavior looks like.
Open Claude Code in the repository and try one or two prompts such as:

- "Add `polars` as a dependency"
- "Create and run a Polars script that groups sales by city"

If the configuration is working, Claude should prefer commands like:

- `uv add polars`
- `uv run python <script>.py`

If it reaches for `pip install` or plain `python`, the repository instructions
are not specific enough yet.

## Use the Bundled `CLAUDE.md`

Instead of writing a new file from scratch, start from the bundled
[CLAUDE.md](/Users/peterfulop.me/code/coding-agents-uv-setup/.claude/CLAUDE.md).
It already defines a strict `uv` workflow, command patterns, testing and
linting entrypoints, and the hard constraints that keep Claude out of `pip`
and manual environment management.

For a single repository, copy that file to the project root as `CLAUDE.md`.

If you already generated a starter file with `/init`, replace the Python
instructions with the version from the bundled file rather than merging the two
by hand.

## What the Included File Covers

The bundled [CLAUDE.md](/Users/peterfulop.me/code/coding-agents-uv-setup/.claude/CLAUDE.md)
already gives Claude Code the important signals:

- `uv` is the only supported Python package manager
- all execution goes through `uv run`
- dependencies are changed with `uv add` and `uv remove`
- testing, Ruff, and type checks use `uv run ...`
- manual virtualenv handling and direct `pip install` are forbidden

That means most users should copy the file as-is, then make only small
project-specific edits if they need extra tools or stricter conventions.

## Common Mistakes

Most configuration problems come from instructions that are technically true
but too vague. Watch for these patterns:

- mentioning `uv` without saying it is the required tool
- telling Claude how to install packages but not how to run scripts
- allowing `python`, `pip`, and `uv` to appear side by side without a clear
  preference
- forgetting that tool runs should also go through `uv run`
- trimming down the bundled `CLAUDE.md` so aggressively that key rules disappear

It also helps to start from the bundled file and edit less, not more.

## Example Test With Polars

If you want a concrete smoke test, use a task that forces Claude to both add a
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
Claude edits dependency files manually or uses `pip install`, the instructions
still need work.

## Optional: Make It Global

If you want the same default behavior across multiple repositories, use the
bundled [CLAUDE.md](/Users/peterfulop.me/code/coding-agents-uv-setup/.claude/CLAUDE.md)
as the base file for your user-level `CLAUDE.md` as well:

- macOS/Linux: `~/.claude/CLAUDE.md`
- Windows: `%USERPROFILE%\.claude\CLAUDE.md`

Global defaults are convenient, but repository-level instructions are still the
better place for project-specific rules. A practical pattern is to reuse this
file broadly, then add only small repository overrides where needed.

