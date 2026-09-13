# Bootstrap Benchmark Suites

Use this reference when creating the first agentic benchmark suite for a Python library, or when expanding a suite after new public APIs and docs are added. The goal is to benchmark whether an agent can behave like a competent library user: find the intended docs or skill, copy the right pattern, write a small script, and validate it without reading internals or changing the library.

## Discover Library Use Cases

Survey user-facing material before reading implementation internals. Prioritize sources in this order:

1. `README.md`, project landing pages, and quickstarts. These define the workflows the library publicly promises.
2. `docs/`, `doc/`, Sphinx `conf.py`, MyST notebooks, generated API pages, and tutorial pages. Look for multi-step workflows and configuration gotchas.
3. `examples/`, `notebooks/`, `tutorials/`, `demo/`, and benchmark examples. These often reveal the intended order of calls better than API references.
4. Tests that exercise public imports. Search for package-root imports, fixture-free usage, integration tests, smoke tests, and examples embedded in tests.
5. Public package exports such as `__all__`, `py.typed`, entry points, CLI commands, and documented submodules. These define the allowed public surface.
6. Public docstrings on exported classes and functions. Use summaries, parameters, returns, warnings, and examples to identify standalone tasks.
7. Packaging metadata and optional dependency groups. These reveal extras such as `dev`, `docs`, `bench`, GPU, database, plotting, or backend integrations that need setup commands.
8. Changelog, migration notes, deprecation warnings, and troubleshooting sections. Convert recurring gotchas into trace rules and task prompt boundaries.

Avoid deriving benchmarks primarily from private source internals. If the only way to discover a workflow is private code spelunking, improve the user skill or docs first, then benchmark it.

## Choose Examples To Create

Create benchmark tasks for workflows that are high-value, deterministic, and small enough for one file. Good tasks usually combine two or three public APIs in the order a user needs them.

Prefer examples that cover:

- Quickstart path: minimal object construction and one useful operation.
- Data modeling: build valid inputs, schemas, tensor dictionaries, config objects, or request payloads.
- Core algorithm/model path: instantiate the main class or function and run it on toy data.
- Backend or optional integration path: select a backend, extra dependency, storage engine, plotting adapter, or accelerator mode.
- Persistence or IO path: load/save, serialize/deserialize, export/import, or round-trip conversion.
- Inspection path: read shapes, metrics, predictions, summaries, diagnostics, or explainability outputs.
- Error-prone boundary: documented constraints such as shapes, dtypes, import paths, indexing conventions, batching rules, or unsupported combinations.
- Compositional workflow: a realistic sequence that joins APIs documented in separate topics.

Defer examples that require network access, large datasets, credentials, long training runs, fragile wall-clock assertions, non-deterministic services, or private APIs. If such behavior is central to the library, mock the external boundary or use tiny local fixtures.

Each task should be answerable by reading the user skill and packaged docs. If an agent would need to inspect implementation source, add a docs example or API card before adding the benchmark.

## Task Design Rules

Write tasks as user requests, not implementation puzzles.

- Require a single output script, notebook, or config file under `solution_dir`.
- Use explicit paths in both `prompt` and `allowed_paths`.
- State that only documented public APIs should be used.
- Include enough acceptance criteria for deterministic validation.
- Validate behavior by running the produced artifact, not by only checking text.
- Prefer assertions over prints; prints are useful only as an additional success marker.
- Keep setup commands deterministic and local to the worktree.
- Add optional dependency installation to `setup_commands` when the base project environment does not include it.
- Keep validation fast enough for repeated runs; target seconds, not minutes.

## Suite Config Contents

Every suite config should contain:

- `suite_name`: human-readable benchmark suite name.
- `library_name`: name used in prompts, reports, and judge context.
- `benchmark_dir`: directory containing task YAML files.
- `output_dir`: directory for results, traces, reports, worktrees, and SQLite index.
- `solution_dir`: directory agents may write benchmark solutions into.
- `agent_name`: benchmark agent name to place in the isolated opencode config.
- `judge_agent_name`: judge agent name to place in the isolated opencode config.
- `agent_prompt`: system prompt that instructs the agent to write the required file first, validate, iterate on failures, prefer user docs/skills, and avoid source internals.
- `judge_prompt`: system prompt that requires structured, rubric-only judgement.

Add these when applicable:

- `user_skill_path`: copy an already-present skill directory into each isolated worktree.
- `user_skill_install_command`: run the library's installer inside each worktree, for example `python -m mypkg.agent_docs install-opencode-skill --project-root .`.
- `trace_rules_path`: JSON rules for deterministic failure signals.
- `db_path`: custom SQLite location when multiple suites share an output area.

Use `user_skill_install_command` when benchmarking the packaged library docs exactly as users will receive them. Use `user_skill_path` when iterating on a local unpublished skill copy.

## Task YAML Contents

Every task file should contain:

- `id`: stable machine-readable ID.
- `name`: short human-readable name.
- `description`: what workflow is being tested.
- `timeout_seconds`: task-specific opencode timeout.
- `tags`: topic, API area, dependency, backend, or difficulty labels.
- `allowed_paths`: exact solution files or narrow globs under `solution_dir`.
- `setup_commands`: deterministic shell commands needed before the agent runs.
- `validation_commands`: commands that prove the generated artifact works.
- `prompt`: user-facing request with exact output path and public-API boundary.
- `judge_rubric`: scoring criteria aligned with the validation and intended workflow.
- `minimum_judge_score`: threshold for LLM-judge pass/fail.

## Trace Rule Contents

Trace rules should capture repeated, deterministic failure modes discovered during suite design and full runs. Useful rule categories include:

- Wrong imports: package-root imports for APIs that live in submodules, renamed symbols, private modules, or deprecated aliases.
- Source spelunking: repeated reads of implementation files when docs or the user skill should have answered the question.
- Shape and dtype mistakes: known tensor ranks, batch axes, index dtypes, or schema keys.
- Unsupported combinations: mutually exclusive options, backend limitations, unavailable extras, or non-portable behavior.
- Validation failures: missing expected output file, edits outside `allowed_paths`, missing dependency setup, or long-running commands.

Each rule should include a precise `label`, a specific `regex`, the narrowest useful `corpus` (`solution`, `assistant`, `validation`, or `all`), and an actionable `remediation` that tells maintainers which docs, examples, imports, or task wording to fix.

## Bootstrap Workflow

1. Inventory use cases from docs, examples, tests, exports, docstrings, and optional dependency metadata.
2. Map each use case to the user-skill topic or docs page that should teach it.
3. Create or update the user skill so every benchmarked workflow has a copy-pasteable example or API card.
4. Create one task YAML per workflow, with deterministic validation and narrow `allowed_paths`.
5. Create suite config with `user_skill_path` or `user_skill_install_command` and `trace_rules_path`.
6. Run the suite with `--dry-run --html` to validate paths, config parsing, task loading, and report generation.
7. Run one real repetition with `--preserve-worktrees` while designing tasks.
8. Inspect traces and convert repeated failures into docs examples, import rules, task prompt fixes, or trace rules.
9. Re-run the suite and compare pass rate, token use, tool calls, and source-spelunking frequency.

## Example Discovery Searches

Use repository search tools rather than reading files randomly. Useful patterns include:

- File discovery: `README*`, `docs/**/*`, `examples/**/*`, `notebooks/**/*`, `tutorials/**/*`, `tests/**/*`, `src/**/*.py`, `<package>/**/*.py`.
- Export discovery: `__all__`, `entry_points`, `[project.scripts]`, `console_scripts`, `py.typed`.
- Example discovery: fenced Python blocks, `pytest.mark`, `doctest`, `if __name__ == "__main__"`, notebook code cells.
- Gotcha discovery: `deprecated`, `warning`, `raises`, `ValueError`, `NotImplemented`, `TODO`, `NOTE`, `must`, `cannot`, `shape`, `dtype`, `backend`.

Turn each search result into either a benchmark task, a user-skill docs improvement, or a trace rule. Do not add benchmarks for workflows the library does not publicly document or support.
