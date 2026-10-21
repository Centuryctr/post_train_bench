# 实验摘要：qwen17_aime2025_10h_max_r02

## 基本信息

- 基础模型：Qwen/Qwen3-1.7B-Base
- 下游任务：AIME 2025（30 题）
- Agent：gpt-6.1-sol，reasoning effort=max
- 时间预算：10 小时
- 状态：完整结束并完成控制器 final_full 复评

## 正式结果

控制器统一评测中，基础模型为 1/30（3.33%），最终模型为 3/30（10.00%），净提升 2 题、6.67 个百分点。Agent 内部曾在 16,000 token、6 并发设置下测得所选模型 5/30（16.67%）；该分数高于控制器复评，因此正式对外结果应采用 3/30，并将 5/30 仅视为内部选择证据。

## 方法与效果

| 方法 | 观测结果 | 判断 |
|---|---|---|
| 多阶段全参数 SFT | 使用 2025 年考试前的数学推理数据、历史竞赛题和答案格式训练；多数阶段在 0–3/30，单靠继续 SFT 的收益不稳定 | 有助于格式和基本能力，但不是最终主要增益来源 |
| 精确整数奖励 GRPO | 多个 RL 分支达到 2–4/30；最佳 RL 候选约 4/30，未超过最终 self-RFT 候选 | 有一定提升，但未入选最终模型 |
| 本地生成并验答案的 self-RFT | self_rft_high/checkpoint-128 在 agent 内部完整评测中达到 5/30，为内部最佳 | 选入最终模型 |
| 参数平均 | self_rft_interpolate50 为 4/30，低于所选模型 5/30 | 变差，未采用 |
| 纯答案格式、短链和部分 LoRA 分支 | 多个分支仅 0–3/30，部分模型出现过长输出或不能正常结束 | 变差或无明确收益 |
| 解码参数调整 | 不同轮次约 2–5/30，波动明显 | 小样本下不稳定 |

## 最终选择

最终权重来自 self_rft_high/checkpoint-128：在已有完整内部试验中分数最高，同时通过模型文件、数据来源和受保护评测文件校验。独立的 AIME 2024 检查为 6/30，但其数据集和生成设置不同，不能与 AIME 2025 直接比较。

## 解释边界

AIME 只有 30 题，单次得分方差很大；实验又多次使用 AIME 2025 聚合分数做模型选择，因此这些分数更接近开发集结果。方法级“提升/下降”依据 agent 的同设置候选比较；最终净收益以控制器 base_full 和 final_full 为准。

## 主要证据

- controller/base_full/metrics.json
- controller/final_full/metrics.json
- agent_home/task/README.md
- agent_home/task/experiments/RESULTS.md
- agent_home/task/experiments/completion_summary.json
- records/agent_model_outputs.jsonl

