# WebArena baseline trajectories — gpt-5.6-luna

Trajectories from the official WebArena baseline agent (`web-arena-x/webarena`, commit dce0468) on all 812 WebArena tasks.

| Setting | Value |
|---|---|
| Agent | Official prompt-based agent, `p_cot_id_actree_2s.json` (CoT, accessibility tree, 2-shot) |
| Agent model | `openai/gpt-5.6-luna` via OpenRouter |
| Judge (fuzzy_match) | `openai/gpt-5.6-luna` |
| Max steps | 30 |
| Result | **125 / 810 scored tasks = 15.4%** (2 tasks not scored: harness crash) |

## Layout
- `luna_webarena_baseline/trajectories/render_<task_id>.html`: one page per task. Open it in a browser to see every step: observation, full model response, parsed action.
- `luna_webarena_baseline/results.csv`: task id, sites, PASS / FAIL / NOT_SCORED, log file, intent.
- `luna_webarena_baseline/logs/`: the 8 run logs.
- `luna_webarena_baseline/REPORT_BASELINE.md`: per-site and per-evaluator breakdown.
- `luna_webarena_baseline/config.json`, `error.txt`: run configuration and error log.

The self-hosted site address is replaced by `<EC2_HOST>` throughout. Playwright trace recordings are not included (7.3 GB).
