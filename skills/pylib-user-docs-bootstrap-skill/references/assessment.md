# Assessment

Read this first when deciding whether to add a dedicated bootstrap pass or use `pylib-user-skill` directly.

## Rules

- Use this bootstrap skill when the target Python library does not yet have a packaged agent skill, docs retrieval CLI, generated API index, installer, and tests.
- Use `pylib-user-skill` directly when those features already exist and the task is to improve completeness, examples, drift checks, or benchmark performance.
- Keep bootstrap separate from the main skill because the work is operational: discover repo shape, add files, avoid import traps, and make a minimal version pass tests.
- Keep `pylib-user-skill` focused on the destination contract: topic granularity, example coverage, shapes, gotchas, boundaries, verification loop, and benchmark-driven refinement.
- Do not let bootstrap become a manual documentation sprint; its goal is a working skeleton with high-value examples for the primary workflows.

## Necessity Decision

Create the bootstrap skill when at least two of these are true:

- There is no `agent_docs.py` or equivalent retrieval CLI.
- There is no packaged `agent_skill/SKILL.md` inside the installable package.
- API docs are handwritten only, with no generated `api_index.json` freshness guard.
- Existing examples are notebooks or README snippets but not extractable runnable code blocks in topic docs.
- Importing the package root can fail without optional dependencies, making docs tooling fragile.
- There are no tests for skill installation, search output, or API-index freshness.

## Bootstrap Outcome

After this skill is applied, `pylib-user-skill` should have something concrete to audit and improve: a real docs skill that installs, retrieves, searches, and reports stale API metadata.
