# Repo Discovery

Read this before writing files in an unfamiliar Python library repo.

## Rules

- Inspect git status first and do not overwrite unrelated user changes.
- Detect whether the package layout is flat (`<pkg>/`) or src-based (`src/<pkg>/`); do not assume `src/`.
- Read `pyproject.toml`, `setup.cfg`, or `setup.py` to identify package names, package-data behavior, test tooling, and Python version constraints.
- Read package `__init__.py` and public docs to identify public exports and import rules.
- Search for public classes/functions, but prefer existing `__all__` when it is maintained.
- Identify heavy import-time dependencies by trying a minimal package import in the available interpreter.
- Use `python3` if `python` is unavailable in the environment; do not hard-code either in docs unless the repo already standardizes on one.
- Check existing tests before adding new ones so docs tests match the repo's conventions.

## Discovery Checklist

- Package directory: flat or `src`.
- Public package name and import module.
- Existing `__all__`, root exports, submodule exports, and intentionally non-root exports.
- Existing user-facing workflows from README, docs, notebooks, or tests.
- Runtime dependency hazards: ML frameworks, database drivers, cloud SDKs, compiled extensions, optional extras.
- Build inclusion: whether package data under `agent_skill/` will be included by current packaging config.
- Test command that can run without optional heavy dependencies if possible.

## Dependency Trap Example

If `python3 -m <pkg>.agent_docs list` imports `<pkg>.__init__` and `__init__` eagerly imports a missing optional dependency, the docs CLI is not zero-dependency. Fix by either making root exports lazy when safe or writing tests/imports that load `agent_docs.py` without triggering the heavy root path.
