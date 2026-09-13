---
name: pylib-agent-benchmarking
description: Use when benchmarking opencode agents on Python-library usage tasks, especially to measure whether agent-facing docs or skills improve correctness, tool use, and token efficiency.
---

# Python Library Agent Benchmarking

Use this skill to run repeatable agentic coding benchmarks for Python libraries. The benchmarked agent should act as a library user: create a small standalone application/example file, validate it, and avoid modifying the library internals. The runner creates isolated git worktrees, invokes `opencode run`, records token/tool metadata when available, validates the patch, optionally runs an LLM judge, analyzes traces with configurable rules, and writes JSONL/CSV/HTML reports.

## Quick Start

Create or copy a suite config and task YAMLs, then run a dry run first:

```bash
python skills/pylib-agent-benchmarking/scripts/run_agent_benchmarks.py \
  --suite-config skills/pylib-agent-benchmarking/suite.example.yaml \
  --dry-run \
  --html
```

Run real benchmarks with a local or remote opencode model:

```bash
python skills/pylib-agent-benchmarking/scripts/run_agent_benchmarks.py \
  --suite-config path/to/suite.yaml \
  --model anthropic/claude-sonnet-4-6 \
  --parallelism parallel \
  --max-workers 4 \
  --repetitions 3 \
  --html
```

## Suite Config

The suite config is intentionally small and portable:

```yaml
suite_name: MyLib API Usage Benchmarks
library_name: MyLib
benchmark_dir: benchmarks
output_dir: agent_benchmark_results
solution_dir: agent_benchmark_solutions
user_skill_path: .opencode/skills/mylib-user-guide
trace_rules_path: trace-rules.json
agent_prompt: |
  You are a careful benchmarked coding agent. Write the required solution file first,
  then validate with the requested commands and iterate on failures. Prefer the
  library's user-facing docs and opencode skill over reading source internals.
judge_prompt: |
  You are an impartial evaluator. Return only the requested structured JSON judgement.
```

Paths are resolved relative to the suite config file unless absolute.

## Task YAML

Each file in `benchmark_dir` defines one benchmark:

```yaml
id: quickstart-usage-script
name: Quickstart Usage Script
description: Create a small script that uses the documented public API.
timeout_seconds: 300
tags:
  - quickstart
  - library-usage
allowed_paths:
  - agent_benchmark_solutions/quickstart_usage.py
setup_commands: []
validation_commands:
  - python agent_benchmark_solutions/quickstart_usage.py
prompt: |
  You are working as a user of MyLib. Create a standalone script at
  agent_benchmark_solutions/quickstart_usage.py using only documented public APIs.
judge_rubric: |
  Score from 1 to 5. Award high scores for runnable public API usage, meaningful
  assertions, and no library source edits.
minimum_judge_score: 4
```

## Metrics Collected

- Wall-clock duration.
- Input, output, total tokens, and cost when provider metadata exposes them.
- Tool call count, counts by tool, and ordered tool call trace.
- Model, judge model, opencode version, suite version, task hash, commit hash, dirty status, and runner configuration.
- Validation stdout/stderr, opencode event stream, optional session export, patch diff, trace bundle, trace analysis, and judge output.
- JSONL, CSV, `summary.json`, `report-data.json`, optional `index.html`, and a local SQLite index.

## Inspect Results

Start with the generated summary and HTML report:

```bash
python -m json.tool agent_benchmark_results/summary.json
open agent_benchmark_results/index.html
```

Browse the SQLite index with the bundled helper:

```bash
python skills/pylib-agent-benchmarking/scripts/browse_benchmark_db.py \
  --db-path agent_benchmark_results/agent_benchmarks.sqlite recent --limit 20
python skills/pylib-agent-benchmarking/scripts/browse_benchmark_db.py \
  --db-path agent_benchmark_results/agent_benchmarks.sqlite remediation
python skills/pylib-agent-benchmarking/scripts/browse_benchmark_db.py \
  --db-path agent_benchmark_results/agent_benchmarks.sqlite artifacts <run-id>
```

Regenerate trace analysis after changing trace rules:

```bash
python skills/pylib-agent-benchmarking/scripts/inspect_agent_traces.py \
  agent_benchmark_results \
  --trace-rules path/to/trace-rules.json
```

Each run stores `opencode-events.jsonl`, optional `session-export.json`, `patch.diff`, validation output, `trace.json`, `trace-analysis.json`, and `judge.json`. Use `browse_benchmark_db.py artifacts <run-id>` to locate them.

## Trace Rules

Use `trace_rules_path` to add library-specific deterministic failure signals without editing the runner:

```json
[
  {
    "label": "package-root import used for non-exported class",
    "regex": "from\\s+mylib\\s+import\\s+.*\\bClient\\b",
    "corpus": "solution",
    "remediation": "Document exact import paths in the user skill and quickstart examples."
  }
]
```

Supported `corpus` values are `solution`, `assistant`, `validation`, and `all`.

## Modular Extension Points

- Benchmark execution lives in `scripts/run_agent_benchmarks.py`.
- Read-only result inspection lives in `scripts/benchmark_results.py`; swap or wrap this module for richer analytics without changing execution.
- The SQLite browser CLI is `scripts/browse_benchmark_db.py`; replace it with a dashboard or notebook if needed.
- The trace re-analysis CLI is `scripts/inspect_agent_traces.py`; trace rules are external JSON so library-specific signals stay out of runner code.
- `report-data.json` is the stable frontend payload. The bundled `index.html` is a minimal default and can be replaced by a higher-quality frontend that consumes the same JSON.

## Workflow

- Start with one deterministic task that validates with `python path/to/script.py`.
- Keep generated solutions under one directory and put only that directory in `allowed_paths`.
- Add one benchmark for every public API workflow that the user skill claims to teach.
- Run at least one repetition with `--preserve-worktrees` while designing a new task.
- Use trace analysis to convert repeated failure modes into docs examples, import rules, and unsupported-boundary notes.

## Safety

The runner uses `opencode run --auto` by default, but every run happens in a detached temporary git worktree with a restricted benchmark-specific opencode config. Use `--no-auto` for interactive debugging.

## Verification

After changing a suite or this skill, run:

```bash
python skills/pylib-agent-benchmarking/scripts/run_agent_benchmarks.py --suite-config path/to/suite.yaml --dry-run --html
```
