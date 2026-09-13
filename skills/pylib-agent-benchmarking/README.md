# Python Library Agent Benchmarking

Standalone opencode skill for benchmarking agentic coding performance on Python-library usage tasks.

The benchmark runner is generic: tasks, docs-skill installation, output paths, agent prompts, and trace-analysis rules are configured by suite files rather than hard-coded for one library.

Result inspection is modular. `scripts/benchmark_results.py` exposes read-only SQLite helpers, `scripts/browse_benchmark_db.py` provides a small CLI, `scripts/inspect_agent_traces.py` regenerates trace analysis, and `report-data.json` is emitted for custom dashboards or richer frontends.

See `SKILL.md` for usage, `references/bootstrap-suite.md` for use-case discovery and suite bootstrapping guidance, and `suite.example.yaml` plus `benchmarks/example-task.yaml` for a minimal suite.
