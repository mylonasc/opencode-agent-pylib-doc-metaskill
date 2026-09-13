# Dependency-Light CLI

Read this before implementing `python -m <pkg>.agent_docs`.

## Rules

- The retrieval CLI must use only the Python standard library.
- Do not import library runtime modules from `agent_docs.py`; read static packaged resources instead.
- Use `importlib.resources.files(<pkg>.agent_skill)` for installed resources.
- If tests load `agent_docs.py` by file path to avoid package-root dependency imports, provide a filesystem fallback to the sibling `agent_skill/` directory.
- Keep search bounded and paged; default to no more than 10 hits per page.
- Print structured API cards before snippet hits when an API index entry matches.
- Make installer overwrite-safe: refuse when destination exists unless `--force` is provided.
- Installer destination should be `.opencode/skills/<skill-name>/` under the selected project root.

## Commands

The CLI should expose:

```bash
python -m <pkg>.agent_docs list
python -m <pkg>.agent_docs get <topic>
python -m <pkg>.agent_docs get <topic> --examples
python -m <pkg>.agent_docs get <topic> --rules
python -m <pkg>.agent_docs search "<regex>" --limit 10 --page 1 --context 2
python -m <pkg>.agent_docs search "<regex>" --examples-only
python -m <pkg>.agent_docs install-opencode-skill --project-root .
python -m <pkg>.agent_docs install-opencode-skill --project-root . --force
```

## API Card Shape

Search output should include enough information for an agent to write code without opening source:

```text
API <name>
  import: from <module> import <name>
  signature: <name>(...)
  summary: <one-line docstring summary>
  returns: <returns note or not documented>
  topic: <topic>
  covering_example: <topic or none>
```

## Import Hazard Pattern

If package-root imports are heavy, this command should still work:

```bash
python -m <pkg>.agent_docs list
```

If it does not, make the root package import safe for metadata-only modules when that is backward-compatible, or document and test a direct module-loading fallback for the docs tests.
