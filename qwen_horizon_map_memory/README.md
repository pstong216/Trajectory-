# horizon harness — qwen3.8-27b, WebArena map, with memory

All 109 WebArena map tasks, run with the `long-horizon-webagent` harness (branch `working/no-gating`) with cross-task memory on.

| Setting | Value |
|---|---|
| Planner / executor | `qwen/qwen3.8-27b` via OpenRouter (planner: temperature 1.0, reasoning medium; executor: temperature 0, no reasoning) |
| Judge (fuzzy_match) | `openai/gpt-5.6-luna` |
| Memory | per-website task-completion log (TCM) + writable site knowledge store, shared across the 109 tasks |
| Completion gate | off |
| Limits | 30 browser steps, 7200 s per task |
| Result | **51 / 109 = 46.8%** (same harness without memory: 53 / 109 = 48.6%) |

## Layout
- `tasks/<task_id>/task.log`: human-readable trajectory: each planner step's subgoal, tools called, executor action and outcome.
- `tasks/<task_id>/planner.jsonl`, `executor.jsonl`: every model call (prompt summary, reasoning, output, tool calls).
- `tasks/<task_id>/env_steps.jsonl`, `trajectory.jsonl`, `episode.jsonl`: browser actions, observations, and the episode record incl. the final evaluator output.
- `tasks/<task_id>/result.json`, `metrics.json`: score and per-task metrics.
- `metrics_tasks.csv`, `results.json`, `metrics_summary.json`, `site_results.json`: run-level results.

When a task was attempted more than once (resume after an infrastructure failure), the completed attempt is included.
