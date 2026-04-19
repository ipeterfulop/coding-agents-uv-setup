# Python Repo Health Criteria

Use these criteria to turn repo evidence into a compact pass/warn/fail report.

## Repo Summary

Extract these basics from `pyproject.toml` and nearby files:

- project name
- `requires-python`
- package/app intent if obvious
- runtime dependencies
- dependency groups or dev dependencies if present
- configured tools: `ruff`, `pytest`, `coverage`, `ty`
- lockfile and CI file presence

## Check Matrix

Evaluate at least these checks:

| Check | Pass | Warn | Fail |
|---|---|---|---|
| Lockfile | `uv lock --check` exits 0 | repo appears uv-based but lock expectations are unclear | `uv lock --check` fails or uv lockfile is expected but missing |
| Lint | `uv run ruff check .` exits 0 | ruff is not wired or command cannot run because tooling is missing | command runs and reports violations |
| Test collection | `uv run pytest --collect-only` exits 0 | pytest is absent or no test posture is defined | command runs and collection errors occur |
| Types | `uv run ty check` exits 0 | `ty` posture is absent or tool is not wired | command runs and reports type errors |
| CI alignment | CI runs the same core stack or a documented equivalent | CI exists but misses one or more core checks | no CI, or CI clearly diverges from declared local workflow |

## Dependency Policy

Look for dependency hygiene signals:

- runtime vs dev-only separation
- direct URL, VCS, or path dependencies
- obviously broad or floating requirements for critical deps
- duplicate tooling across runtime dependencies and dev groups
- missing lockfile despite uv usage

Do not over-enforce version pinning heuristics. The goal is repo health, not dogma. Flag only the patterns that create operational ambiguity.

## Coverage Policy

Treat coverage as a policy question, not just a test question.

Pass:
- explicit coverage configuration exists and a floor is enforced in CI or test commands

Warn:
- tests exist but no coverage threshold is defined
- coverage is configured locally but not enforced in CI

Fail:
- a stated coverage floor exists but CI or commands clearly violate it

Common evidence:
- `[tool.coverage.*]` in `pyproject.toml`
- `pytest` addopts with coverage gating
- CI workflow flags like `--cov-fail-under=...`

## Type-Check Posture

Pass:
- `ty` is configured and run locally and/or in CI
- exclusions or gradual adoption boundaries are explicit

Warn:
- `ty` is not configured and the repo gives no explicit stance
- type checking exists only informally or only in one environment

Fail:
- the repo claims a type-check gate but the command currently fails

## CI Config

Inspect whichever automation files exist:

- `.github/workflows/*.yml`
- `.gitlab-ci.yml`
- `.pre-commit-config.yaml`

Healthy CI usually shows:

- Python setup consistent with `requires-python`
- lock/install path compatible with uv
- lint, test, and type checks represented
- reasonable ordering and failure behavior

## Result Priorities

Prioritize findings in this order:

1. broken or missing lockfile invariants
2. CI and local-tooling drift
3. failing lint/test/type gates
4. missing coverage/type policy
5. dependency hygiene concerns
