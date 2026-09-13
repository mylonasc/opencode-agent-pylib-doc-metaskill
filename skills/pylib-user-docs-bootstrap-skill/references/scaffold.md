# Scaffold

Read this when adding the initial files for a library-specific user docs skill.

## Rules

- Put shipped skill resources inside the importable package so `importlib.resources` can find them after installation.
- For flat layout, use `<pkg>/agent_skill/` and `<pkg>/agent_docs.py`.
- For src layout, use `src/<pkg>/agent_skill/` and `src/<pkg>/agent_docs.py`.
- Keep `agent_skill/SKILL.md` short; it is an index and router, not the full manual.
- Start with topics based on agent tasks, not source modules.
- Include at least `quickstart` and one topic per major public workflow.
- Add a generator script outside the package, usually `scripts/generate_agent_api_index.py` or `tools/generate_agent_api_index.py`.
- Commit generated `api_index.json` beside `SKILL.md`.

## Minimum Layout

```text
<pkg>/
|-- agent_docs.py
`-- agent_skill/
    |-- __init__.py
    |-- SKILL.md
    |-- api_index.json
    `-- references/
        |-- quickstart.md
        |-- data.md
        |-- models.md
        `-- backends.md
scripts/
`-- generate_agent_api_index.py
tests/
`-- test_agent_docs.py
```

## Topic Shape

Each topic should use this shape:

````markdown
# Topic Title

Read this when ...

## Rules

- State exact import paths, required shapes, ID conventions, lifecycle rules, and unsupported boundaries.

## Workflow Example

```python
# Runnable, copy-pasteable example with assertions.
```
````

## API Index Fields

Generate entries with:

- `name`
- `import`
- `signature`
- `summary`
- `returns`
- `source`
- `topic`
- `example`

Use ordered heuristic topic tagging. Examples: `query` names go to query docs, `sample` names go to sampling docs, `ingest` names go to ingestion docs, backend/open/create names go to backend docs, model/layer names go to model docs, and default goes to quickstart.
