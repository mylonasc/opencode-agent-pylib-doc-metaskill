---
name: pylib-user-docs-bootstrap-skill
description: Use when bootstrapping a Python library repo with the basic packaged agent-docs features expected by pylib-user-skill, including skill resources, retrieval CLI, API index, installer, and tests.
---

# Python Library User Docs Bootstrap Skill

Use this skill when a Python library does not yet have agent-facing documentation infrastructure and needs a first working implementation before applying the full `pylib-user-skill` quality loop.

## Assessment

Keep this separate from `pylib-user-skill`. The main skill is the target-state contract for high-quality shipped docs; this bootstrap skill is the initial repo-modification workflow for discovering package layout, scaffolding files, avoiding dependency traps, and adding minimum verification.

## Fast Workflow

1. Read `references/assessment.md` to decide whether to bootstrap or only refine existing docs.
2. Read `references/repo-discovery.md` to identify package layout, public API, build config, and import-time dependency hazards.
3. Read `references/scaffold.md` to add `agent_skill/`, `agent_docs.py`, a generator script, and first-pass topics.
4. Read `references/dependency-light-cli.md` before writing the retrieval CLI.
5. Read `references/minimum-tests.md` before running verification.
6. Read `references/handoff.md` to leave the repo ready for `pylib-user-skill` and benchmarking.

## Bootstrap Deliverables

- A packaged skill directory under the importable library package, usually `<pkg>/agent_skill/` or `src/<pkg>/agent_skill/`.
- A short `SKILL.md` router with frontmatter, retrieval commands, topics, import rules, usage rules, boundaries, and one minimal runnable pattern.
- Topic references with `## Rules` and runnable `python` code blocks.
- A zero-extra-dependency retrieval CLI, usually `python -m <pkg>.agent_docs`.
- Commands for `list`, `get <topic> [--examples|--rules]`, `search <regex> [--limit --page --context --examples-only]`, and `install-opencode-skill`.
- A generated `api_index.json` with public API names, imports, signatures, summaries, returns notes, source locations, topics, and covering examples.
- A freshness check that fails when the committed API index differs from a fresh build.
- Focused tests for resource presence, topic extraction, search/API cards, installer overwrite behavior, and index freshness.

## Operating Rules

- Bootstrap the smallest useful version; do not attempt complete manual documentation in the first pass.
- Prefer generated API metadata over hand-maintained signatures.
- Keep the docs CLI runnable even when optional runtime dependencies are missing.
- Use exact identifier matching inside runnable examples for coverage; prose mentions do not count.
- If package-root imports require heavy dependencies, avoid importing the package root from docs tests and consider lazy root exports when that is safe.
- After bootstrap passes, switch to `pylib-user-skill` for completeness, drift checks, example execution, and benchmark-driven refinement.

## References

- `assessment`: why this is separate from `pylib-user-skill`, and when not to use it.
- `repo-discovery`: package-layout and public-surface discovery checklist.
- `scaffold`: concrete file layout and minimum content shape.
- `dependency-light-cli`: zero-dependency retrieval and import-time hazard guidance.
- `minimum-tests`: minimum verification before handoff.
- `handoff`: how to transition from bootstrap to full docs and benchmarks.
