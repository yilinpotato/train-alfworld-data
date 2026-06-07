# ALFWorld 训练与评测数据

本仓库整理了 ALFWorld 任务上的训练日志与 API 评测结果，主要包含两部分：

- `任务1 A800/`、`任务2~6 A800/`：A800 训练运行记录。
- `deepseek/`：DeepSeek API 在 ALFWorld 各任务上的评测轨迹与汇总结果。

## 目录结构

```text
.
├── 任务1 A800/
│   ├── run_config.env
│   └── metrics.jsonl
├── 任务2~6 A800/
│   ├── run_config.env
│   └── 任务2~6 A800 metrics.jsonl
└── deepseek/
    ├── alfworld_api_eval/
    └── alfworld_api_eval_full_skills/
```

## 训练日志

训练日志来自 Qwen3-4B-Thinking-2507 在 A800 上的 Skill0 训练实验。

| 目录 | 内容 | 记录数 | 说明 |
| --- | --- | ---: | --- |
| `任务1 A800/` | `metrics.jsonl` | 211 | 任务 1 训练指标 |
| `任务2~6 A800/` | `任务2~6 A800 metrics.jsonl` | 31 | 任务 2~6 训练指标 |

`run_config.env` 保存对应实验配置，包括模型路径、训练数据规模、rollout 配置、学习率、训练步数、显存参数和 SkillZero 配置等。

`metrics.jsonl` 每行是一条训练 step 记录，主要字段包括：

- `step`：训练步数。
- `metrics`：该步的指标字典。
- `episode/success_rate`：当前 rollout 的成功率。
- `episode/reward/mean`：平均奖励。
- `episode/length/mean`：平均 episode 长度。
- `actor/*`：actor 训练损失、KL、学习率、梯度等。
- `perf/*`、`timing_s/*`：吞吐、显存、耗时等性能指标。

## DeepSeek 评测数据

`deepseek/` 下包含两组评测：

| 目录 | 说明 | 每类任务 episode 数 |
| --- | --- | ---: |
| `alfworld_api_eval/` | 基础 prompt / 技能设置下的评测 | 100 |
| `alfworld_api_eval_full_skills/` | full skills 设置下的评测 | 20 |

每个任务目录包含：

- `*.jsonl`：逐 episode 轨迹；每行是一条 episode。
- `*.json`：同批评测的完整 JSON 结果。
- `*.summary.json`：单任务汇总指标。
- `alfworld_api_eval.state.json`：评测状态文件。
- `alfworld_api_eval.overall_summary.json`：该组评测的全任务汇总。

单条 episode 记录的核心字段：

- `episode`、`env_index`：episode 编号和环境编号。
- `task_type_id`、`task_type`：任务类型。
- `steps`：逐步交互轨迹。
- `success`、`num_steps`：是否成功和总步数。
- `prompt_tokens_total`、`completion_tokens_total`、`total_tokens`：token 统计。
- `avg_step_latency_s`：平均单步 API 延迟。

`steps` 中每一步包含观测、prompt、模型输出、解析后的环境动作、动作合法性、奖励、完成状态、下一步观测以及 token/延迟统计。

## 任务与结果概览

| 任务 | task_filter | 基础评测成功率 | Full skills 成功率 |
| --- | --- | ---: | ---: |
| `pick_and_place` | `pick_and_place` | 66% | 80% |
| `look_at_obj_in_light` | `look_at_obj_in_light` | 46% | 80% |
| `clean_and_place` | `pick_clean_then_place_in_recep` | 39% | 45% |
| `heat_and_place` | `pick_heat_then_place_in_recep` | 46% | 60% |
| `cool_and_place` | `pick_cool_then_place_in_recep` | 50% | 40% |
| `pick_two_and_place` | `pick_two_obj_and_place` | 52% | 35% |

基础评测使用模型 `deepseek-v4-pro`，每类任务 100 个 episode；full skills 评测同样使用 `deepseek-v4-pro`，每类任务 20 个 episode。

## 文件格式说明

- `.jsonl`：逐行 JSON，适合流式读取或按 episode/step 过滤。
- `.json`：完整评测结果，适合一次性加载分析。
- `.summary.json`：单任务汇总指标。
- `.overall_summary.json`：全任务汇总指标。
- `.env`：训练运行配置，按 `KEY=VALUE` 保存。

