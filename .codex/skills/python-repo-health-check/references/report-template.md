# One-Page Report Template

Use this shape for the final user-facing report.

## Summary line

Start with:

`Overall: pass|warn|fail - one sentence on the current health posture.`

## Repo snapshot

Keep this to 2-4 bullets:

- Python version target
- uv / lockfile presence
- core tool config presence
- CI presence

## Checks

Use a compact table:

| Check | Status | Evidence |
|---|---|---|
| Lockfile | pass/warn/fail | short evidence |
| Ruff | pass/warn/fail | short evidence |
| Pytest collect | pass/warn/fail | short evidence |
| Ty | pass/warn/fail | short evidence |
| CI alignment | pass/warn/fail | short evidence |

## Findings

Limit to the 3-5 highest-value findings. Keep each to one sentence.

Good examples:

- `warn`: `pyproject.toml` declares runtime deps but no dev group for lint/test/type tools, so the repo posture is hard to reproduce consistently.
- `fail`: `uv lock --check` failed, which means the lockfile is out of sync with `pyproject.toml`.
- `warn`: tests are collectible, but there is no explicit coverage floor in config or CI.

## Recommended next actions

Limit to 3 bullets. Make them concrete and ordered.

## Open questions

Include this section only if signals are genuinely missing or contradictory.
