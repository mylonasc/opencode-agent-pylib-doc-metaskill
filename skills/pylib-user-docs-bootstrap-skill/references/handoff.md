# Handoff

Read this after the bootstrap tests pass.

## Rules

- Treat bootstrap as the start of the documentation loop, not the end.
- Run `pylib-user-skill` next to audit topic granularity, import rules, shapes, boundaries, gotchas, example coverage, and drift checks.
- Run `pylib-agent-benchmarking` after the docs install locally and basic retrieval works.
- Convert benchmark failures into topic rules and runnable examples.
- Add examples for every public API an agent may reasonably be asked to use.
- Add shape details where APIs return dicts, tuples, nested records, tensor dictionaries, byte/string IDs, or backend-specific values.
- Add unsupported-boundary notes for every hallucinated capability observed in traces.

## Handoff Checklist

- `python -m <pkg>.agent_docs list` works from a clean checkout with package installed or importable.
- `python -m <pkg>.agent_docs install-opencode-skill --project-root .` installs a real opencode skill.
- `api_index.json` is generated and freshness-checked.
- Tests pass for retrieval, installer safety, API cards, and index freshness.
- The main `SKILL.md` tells agents to write the solution first, validate, iterate, and prefer retrieval over source spelunking.
- Remaining documentation gaps are explicit follow-up items, not hidden assumptions.
