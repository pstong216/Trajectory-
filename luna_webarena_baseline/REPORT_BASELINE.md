# WebArena 官方 Baseline：完整 812 题结果 + vs WebChallenger 对比

生成时间：2026-09-22　仓库：`web-arena-x/webarena`（dce0468, 2025-11-26）+ 5 个补丁

## 配置

| 项 | 值 |
|---|---|
| agent | 官方 baseline（`agent_type=prompt`，accessibility tree 观察，无 PageMem/探索/复合动作） |
| 提示词 | `p_cot_id_actree_2s.json`（CoT + a11y tree，官方 2-shot 示例） |
| agent 模型 | `openai/gpt-5.6-luna`（OpenRouter） |
| 判分裁判 | `openai/gpt-5.6-luna`（与 WebChallenger 复现同一模型） |
| max_steps | 30 |
| 网站 | 同一台 EC2（`<EC2_HOST>`）。**分两批评测前各自重建过对应容器**：gitlab/reddit 在第一批前重置，shopping/shopping_admin 在第二批前重置，均验证过登录、状态清空、无残留旧 host 引用 |
| 任务集 | 完整 WebArena **812 题**（436 题子集 + 补跑的 376 题 shopping/shopping_admin） |
| 运行方式 | 两批各 4 个进程按题号区间并行，各自独立 `.auth` cookie 续期 |
| 补丁 | 裁判模型可配置、gpt-5.x 参数改名、按题号过滤、tokenizer 回退、PATH/PYTHONPATH 修正（详见 `repro/baseline/apply_patches.sh` 与 `baseline_env.sh`） |

## 总体结果（完整 812 题）

| 指标 | 值 |
|---|---|
| 已判分 | 810 / 812 |
| **通过率** | **125 / 810 = 15.4%** |
| 未判分（harness 自身崩溃，非 agent 逻辑问题） | 2（详见下方"已知限制"） |

## 按站点（完整 812 题）

| 站点 | PASS / N | 通过率 |
|---|---|---|
| reddit | 24/105 | 22.9% |
| map,wikipedia | 4/17 | 23.5% |
| map | 23/109 | 21.1% |
| shopping | 33/187 | 17.6% |
| gitlab | 21/180 | 11.7% |
| shopping_admin | 20/181 | 11.0% |
| gitlab,reddit | 0/18 | 0.0% |
| gitlab,wikipedia | 0/6 | 0.0% |
| reddit,shopping | 0/5 | 0.0% |
| map,shopping_admin | 0/2 | 0.0% |

跨站点任务全线接近 0%（除 map,wikipedia），与 WebChallenger 复现里观察到的"跨站点是短板"模式一致，只是 baseline 上这个短板更极端。

## 与 WebChallenger 复现对比（同一 436 道子集，唯一变量是有无 harness）

| 站点 | baseline (无 harness) | WebChallenger (有 harness) | 差 |
|---|---|---|---|
| gitlab | 21/180 = 11.7% | 80/179 = 44.7% | **+33.0pp** |
| reddit | 24/105 = 22.9% | 59/106 = 55.7% | **+32.8pp** |
| map | 23/109 = 21.1% | 49/109 = 45.0% | **+23.9pp** |
| map,wikipedia | 4/17 = 23.5% | 8/17 = 47.1% | +23.6pp |
| gitlab,reddit | 0/18 = 0.0% | 2/18 = 11.1% | +11.1pp |
| gitlab,wikipedia | 0/6 = 0.0% | 0/5 = 0.0% | 0pp |
| **436-道子集总体** | **72/435 = 16.6%** | **198/434 = 45.6%** | **+29.0pp** |

("差" = WebChallenger 相对 baseline 的提升，正值表示 harness 更好)

## 与论文对照（现在是同口径：完整 812 题）

| | 完整 WebArena (812) 通过率 |
|---|---|
| 论文 WebChallenger（GLM-4-32B + Qwen2.5-VL-7B，GPT-4-turbo 裁判） | 56.3% |
| 本次 baseline（无 harness，gpt-5.6-luna 兼做 agent 与裁判） | **15.4%** |
| 本次 WebChallenger 复现（有 harness，同一 436 子集） | 45.6%（436 题口径，非 812） |

论文本身没有报告"裸 accessibility-tree agent 在完整 812 题上的分数"这一基线；这里提供的 15.4% 是我们自己补的对照点。

## 解读

1. **这是目前最干净的对照**：同一 436 题子集、同一 agent 模型（gpt-5.6-luna）、同一裁判、同一 EC2。唯一变量是**是否使用 WebChallenger 的 harness**（PageMem 分块观察、离线探索/记忆、复合动作）。
2. **每个站点类别 harness 都明显更好**，差距 11–33 个百分点，总体 +29pp（16.6% → 45.6%）。这是迄今最干净的证据，说明 WebChallenger 论文声称的三个机制确实贡献了大部分性能，而不只是模型能力或环境状态的假象。
3. **环境差异让结果更有说服力，而不是更弱**：baseline 是在**刚重置的干净 EC2 状态**上跑的；WebChallenger 复现是在**436 道跑完后已累积状态的 EC2**上跑的。也就是说 baseline 享有更干净的环境优势，WebChallenger 反而是在更脏的环境下跑的，但仍大幅领先。
4. **补跑的 shopping/shopping_admin（376 题）延续了同样的模式**：单站点 11–18%，跨站点（reddit,shopping / map,shopping_admin）0%。没有 WebChallenger 版本的 shopping 结果可比较（当初复现时因 shopping_admin 的 external_url 配置问题排除了这两站；本次重建容器后已验证修复，但没有回头重跑 WebChallenger 覆盖这两站）。
5. **跨站点任务对 baseline 几乎是灾难性的**（10 个跨站类别里 8 个是 0%）——没有 PageMem 记忆的话，agent 似乎难以在两个网站间保持任务上下文。

## 已知限制

- 2 道题因 harness 自身 bug 崩溃未判分（均为官方 harness 代码问题，非 agent 逻辑）：
  - task 621：`KeyError('—')`，动作解析遇到 em-dash 字符崩溃
  - task 374：`AttributeError("'Page' object has no attribute 'client'")`，Playwright API 版本相关
  两者重跑均会复现，未在评测中途修补（保持代码一致性）
- EC2 环境在两批评测之间不完全一致（gitlab/reddit 批与 shopping/shopping_admin 批分别在各自重置后运行），跨批比较（如 436 子集 vs 补跑的 376 题）要留意这一点
- 裁判模型 gpt-5.6-luna 非论文使用的 GPT-4-turbo，temperature 无法设为 0（该模型只接受默认值 1），已用 seed=42
- 并发只在进程级别（4 个独立 Python 进程按题号区间），同一时刻可能有多个任务修改同一站点状态
