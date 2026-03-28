# Configure Coding Agents to Use `uv`

This repository provides reusable instruction files for coding agents that
should follow a strict `uv`-based Python workflow.

The goal is simple: make the agent choose `uv add`, `uv run`, `uv sync`, and
related `uv` commands by default instead of falling back to `pip`, raw
`python`, ad hoc virtual environments, or manual dependency edits.

## Why `uv`

`uv` is a fast Python package and project manager from Astral. It combines
dependency management, environment management, Python installation, lockfile
generation, and tool execution into one workflow.

In practice, that means fewer moving parts and more consistent agent behavior.
Instead of mixing tools such as `pip`, `virtualenv`, `pip-tools`, or `pipx`,
you can teach the agent one clear command family and expect more predictable
results.

## What This Repository Contains

This repository does not contain an example application. It contains agent-
facing instruction files and companion guides that help you apply the same
`uv` policy in different agent ecosystems.

The two main instruction files are similar in intent and policy, but each is
optimized for its target agent:

- [`.claude/CLAUDE.md`](.claude/CLAUDE.md)
  is for Claude Code.
- [`.codex/AGENTS.md`](.codex/AGENTS.md)
  is for Codex and other tools that read `AGENTS.md`.

They both enforce the same core idea: use `uv` for Python package management
and execution. The difference is the format and phrasing needed for better
interpretability in Claude versus Codex.

## Start Here

If you want to use the Claude-oriented version, start with
[`.claude/README.md`](.claude/README.md).

If you want to use the Codex-oriented version, start with
[`.codex/README.md`](.codex/README.md).

Those two guides explain how to reuse the bundled instruction file in another
repository, what behavior to expect, and how to verify that the agent is
actually using `uv`.

## Repository Structure

| Path | Purpose |
|------|---------|
| [`.claude/CLAUDE.md`](.claude/CLAUDE.md) | Canonical Claude Code instruction file for a `uv`-only Python workflow. |
| [`.claude/README.md`](.claude/README.md) | Guide for making Claude Code use the bundled `CLAUDE.md` and verifying that it prefers `uv`. |
| [`.codex/AGENTS.md`](.codex/AGENTS.md) | Canonical Codex instruction file for a `uv`-only Python workflow. |
| [`.codex/README.md`](.codex/README.md) | Guide for making Codex use the bundled `AGENTS.md` and verifying that it prefers `uv`. |

## Picking the Right File

Use [`.claude/CLAUDE.md`](.claude/CLAUDE.md)
when the agent expects Claude-style project memory.

Use [`.codex/AGENTS.md`](.codex/AGENTS.md)
when the agent expects `AGENTS.md` instructions.

If you support both tools, keep both files aligned on policy, but do not assume
they should be textually identical. They are similar by design, while still
being optimized for Claude and Codex respectively.

## Freeing Up Space

If you use `uv` heavily, its cache directory can grow over time. It is worth
checking occasionally, especially if you frequently install tools or work
across multiple repositories.

To see how much space the cache is using, run:

```bash
du -sh "$(uv cache dir)"
```

If the cache has grown larger than you want, you can remove it with:

```bash
uv cache clean
```

