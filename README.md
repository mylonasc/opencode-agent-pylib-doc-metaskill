# opencode-agent-pylib-doc-metaskill

Skills for creating specialized library user documentation for opencode assistants.

## Skills

- `skills/pylib-user-skill/` — ship a user-assistant skill with a Python library: packaged skill layout, retrieval CLI, auto-generated API index, verification loop, and harness/release discipline. Distilled from the GestaltDB `gestaltdb.agent_skill` + `agent_docs` work, where benchmark-driven documentation cut agent token usage by ~50–78% per task.
- `skills/pylib-agent-benchmarking/` — run reusable opencode agentic coding benchmarks for Python-library usage tasks: isolated git worktrees, task YAMLs, deterministic validation, optional LLM judge, configurable trace rules, token/tool metrics, SQLite/JSONL/CSV/HTML outputs.
