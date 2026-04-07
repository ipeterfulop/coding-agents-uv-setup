# Make Cursor use `uv`

[Cursor](https://cursor.com) rules provide system-level instructions to Agent. They bundle prompts, scripts, and more together, making it easy to manage and share workflows across your team.

_As we have previously create an AGENTS.md file for Codex, we can use it to create a Cursor rule._

In the View -> Command Palette select "File: New Cursor Rule" then in the appearing window run the following command:

```text
/create-rule

Create a Cursor rule based on the file:
../.codex/AGENTS.md

This file defines agent instructions that enforce the use of `uv` for:
- dependency management
- environment setup
- code execution

Ensure all agents follow these constraints consistently.
```

If the rule creation completed successfully, you should see the following message:
```text
Created an always-on Cursor rule at .cursor/rules/python-uv-enforcement.mdc based on .codex/AGENTS.md.

It enforces uv as the only allowed workflow for Python dependency management, environment setup, and code/tool execution across agents by:

- requiring uv add, uv remove, uv sync, uv lock, uv run, and uvx

- forbidding pip, poetry, conda, pipenv, manual virtualenv handling, direct dependency edits, and uv pip install for normal dependency management

- preserving the safety guardrail to prefer project-local [tool.uv] exclude-newer = "14 days"

I also set alwaysApply: true so the rule is consistently available in every session, while the rule text itself scopes enforcement to Python-related tasks. No linter issues were reported.
```


Ensure that once the rule is created, the `alwaysApply: true` is set.

You can download [the rule file from this repository](rules/python-uv-enforcement.mdc).