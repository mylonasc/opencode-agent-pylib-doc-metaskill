# Minimum Tests

Read this before considering bootstrap complete.

## Rules

- Add focused tests for docs infrastructure; do not require full library runtime dependencies unless unavoidable.
- Verify topics can be listed and retrieved.
- Verify `--examples` extracts runnable Python code blocks.
- Verify `search` emits pagination and API cards.
- Verify `install-opencode-skill` creates `.opencode/skills/<skill-name>/SKILL.md`.
- Verify installer refuses overwrite without `--force` and succeeds with `--force`.
- Verify `api_index.json` is fresh against the generator output.
- Verify coverage detection uses exact Python identifiers inside runnable examples, not prose substring matches.
- Run syntax checks on new standard-library tooling.

## Minimum Command Set

Use the repo's preferred test runner, but the bootstrap pass should normally validate these behaviors:

```bash
python3 -m <pkg>.agent_docs list
python3 -m <pkg>.agent_docs search "SomePublicApi" --limit 2
python3 scripts/generate_agent_api_index.py --check
python3 -m py_compile <pkg>/agent_docs.py scripts/generate_agent_api_index.py
python3 -m pytest -q tests/test_agent_docs.py
```

If the repo standardizes on `python` or `uv run`, use that spelling in the repo docs and tests. If the current environment only has `python3`, use `python3` for local validation and mention it in the final report.

## Example Coverage Check

The generator should count a public API as covered only when the exact identifier appears in a fenced `python` code block. This avoids false positives such as `Graph` being counted because prose mentions `GraphNet`.
