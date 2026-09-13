# opencode-agent-pylib-doc-metaskill

Reusable opencode skills for making Python libraries easier for coding agents to use correctly, then measuring whether the documentation actually improves agent performance.

The repository has three complementary skills:

- `skills/pylib-user-docs-bootstrap-skill/` bootstraps a Python library repo with the basic packaged agent-docs features expected by `pylib-user-skill`.
- `skills/pylib-user-skill/` teaches maintainers how to ship a library-specific opencode user skill with their Python package.
- `skills/pylib-agent-benchmarking/` runs agentic coding benchmarks that measure whether agents can use a library's public APIs from documentation, examples, and skills with fewer failures and fewer tokens.

The pattern is intentionally documentation-driven: create focused agent-facing docs, benchmark realistic usage tasks, inspect failures, improve the docs, then rerun the benchmark.

## Why This Exists

General coding agents often waste context reading source internals, guessing import paths, or debugging unsupported usage patterns. A small, task-oriented user skill can make the first solution much more likely to be correct.

This repo captures that loop in reusable form:

1. Ship concise, example-heavy user documentation as an opencode skill.
2. Give agents application-style tasks that require using the public API.
3. Run each task in an isolated git worktree.
4. Validate generated code deterministically.
5. Collect token usage, tool usage, patches, traces, and judge scores.
6. Convert repeated failure modes into documentation fixes.

## Skill: `pylib-user-docs-bootstrap-skill`

Use `skills/pylib-user-docs-bootstrap-skill/` when a Python library does not yet have agent-facing docs infrastructure.

It describes how to add the minimum working features expected by `pylib-user-skill`:

- A packaged `agent_skill/` directory with `SKILL.md`, topic references, and generated `api_index.json`.
- A zero-extra-dependency retrieval CLI such as `python -m <pkg>.agent_docs`.
- `list`, `get`, `search`, and `install-opencode-skill` commands.
- An AST API-index generator with an index freshness check.
- Tests for topic retrieval, API-card search, installer overwrite safety, and freshness.
- Import-time dependency guardrails so docs tooling can run before optional runtime dependencies are installed.

Start here when bootstrapping a repo from no agent docs to a basic, installable, testable docs skill. Then use `pylib-user-skill` for completeness and benchmark-driven refinement.

## Skill: `pylib-user-skill`

Use `skills/pylib-user-skill/` when adding a user-assistant skill to a Python library.

It describes how to package and verify a library-specific skill with:

- A short `SKILL.md` that acts as an index and router, not a full manual.
- Topic references such as quickstart, backends, ingestion, querying, indexing, or sampling.
- Copy-pasteable examples for every public API workflow an agent may be asked to use.
- Exact import rules, return shapes, ID conventions, cleanup rules, and unsupported boundaries.
- A zero-dependency retrieval CLI such as `python -m <pkg>.agent_docs get <topic> --examples`.
- An auto-generated API index for public classes, functions, methods, signatures, and topic tags.
- Tests and drift checks so packaged docs do not silently diverge from code.

Start here when you want a Python package to include its own agent-facing usage guide.

## Skill: `pylib-agent-benchmarking`

Use `skills/pylib-agent-benchmarking/` when measuring whether agents can use a Python library effectively from the available docs and opencode skills.

The benchmark runner provides:

- One temporary git worktree per benchmark run.
- Configurable task YAMLs with prompts, validation commands, setup commands, allowed edit scopes, timeouts, and judge rubrics.
- Optional installation of a target library's opencode user skill into each worktree.
- `opencode run --format json` execution with token, cost, tool-call, and session metadata extraction when available.
- Deterministic validation by running commands such as `python agent_benchmark_solutions/example.py`.
- Allowed-path enforcement so usage benchmarks fail if agents edit library internals.
- Optional LLM-as-judge scoring against a fixed rubric.
- Trace bundles containing events, session export, patch diff, validation output, judge output, and deterministic trace analysis.
- JSONL, CSV, SQLite, `summary.json`, `report-data.json`, and optional HTML outputs.

Start here when you want to compare documentation versions, model configurations, prompts, or opencode skills on realistic public-API usage tasks.

## Quick Benchmark Example

Run the bundled example suite in dry-run mode:

```bash
python skills/pylib-agent-benchmarking/scripts/run_agent_benchmarks.py \
  --suite-config skills/pylib-agent-benchmarking/suite.example.yaml \
  --dry-run \
  --html
```

Run real benchmark repetitions with a model:

```bash
python skills/pylib-agent-benchmarking/scripts/run_agent_benchmarks.py \
  --suite-config path/to/suite.yaml \
  --model anthropic/claude-sonnet-4-6 \
  --parallelism parallel \
  --max-workers 4 \
  --repetitions 3 \
  --html
```

Inspect results:

```bash
python skills/pylib-agent-benchmarking/scripts/browse_benchmark_db.py \
  --db-path agent_benchmark_results/agent_benchmarks.sqlite recent --limit 20

python skills/pylib-agent-benchmarking/scripts/browse_benchmark_db.py \
  --db-path agent_benchmark_results/agent_benchmarks.sqlite remediation

python skills/pylib-agent-benchmarking/scripts/inspect_agent_traces.py \
  agent_benchmark_results \
  --trace-rules path/to/trace-rules.json
```

Use `report-data.json` for custom dashboards or richer frontends. The bundled HTML report is intentionally minimal and replaceable.

## Benchmark Suite Shape

A suite config points the generic runner at library-specific tasks and rules:

```yaml
suite_name: MyLib API Usage Benchmarks
library_name: MyLib
benchmark_dir: benchmarks
output_dir: agent_benchmark_results
solution_dir: agent_benchmark_solutions
user_skill_path: .opencode/skills/mylib-user-guide
trace_rules_path: trace-rules.json
```

Each benchmark task is a YAML file with a prompt and validation command:

```yaml
id: quickstart-usage-script
name: Quickstart Usage Script
allowed_paths:
  - agent_benchmark_solutions/quickstart_usage.py
validation_commands:
  - python agent_benchmark_solutions/quickstart_usage.py
prompt: |
  You are working as a user of MyLib. Create a standalone script at
  agent_benchmark_solutions/quickstart_usage.py using only documented public APIs.
judge_rubric: |
  Score from 1 to 5. Award high scores for runnable public API usage,
  meaningful assertions, and no library source edits.
minimum_judge_score: 4
```

## Recommended Workflow

1. Add or improve the library-specific user skill with `pylib-user-skill` guidance.
2. Write one deterministic benchmark task for each public API workflow the docs claim to teach.
3. Run the benchmark suite and inspect failures with `browse_benchmark_db.py`, `inspect_agent_traces.py`, and per-run trace artifacts.
4. Convert failure patterns into documentation changes: exact imports, runnable examples, shape details, cleanup rules, and unsupported-boundary notes.
5. Rerun the suite and compare pass rates, token usage, tool calls, and remediation actions.

## Modular Design

The benchmarking skill is split so pieces can be replaced later:

- `run_agent_benchmarks.py` handles benchmark execution.
- `benchmark_results.py` provides read-only SQLite inspection helpers.
- `browse_benchmark_db.py` is a small CLI over the SQLite index.
- `inspect_agent_traces.py` regenerates deterministic trace analysis.
- `trace-rules.json` files hold library-specific failure signals outside runner code.
- `report-data.json` is the frontend-neutral payload for higher-quality reports or dashboards.

## Provenance

These skills were distilled from GestaltDB agent-documentation and benchmarking work, where benchmark-driven documentation reduced agent token usage by roughly 50-78% on selected public-API usage tasks.
