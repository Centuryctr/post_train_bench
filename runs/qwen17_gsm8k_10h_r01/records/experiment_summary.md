# 实验摘要：qwen17_gsm8k_10h_r01

## 基本信息

- 基础模型：Qwen/Qwen3-1.7B-Base
- 下游任务：GSM8K（1,319 题）
- Agent：gpt-6.1-sol，reasoning effort=high
- 时间预算：10 小时
- 状态：完整结束并完成控制器 final_full 复评

## 正式结果

控制器 final_full 为 1,136/1,319（86.13%）。该运行目录没有控制器 base_full/metrics.json，因此不能从统一控制器口径计算正式净提升。Agent 报告中的基础模型控制为 300 题上的 32.33%，不是完整 1,319 题评测，不能直接与 86.13% 相减。

## 方法与效果

| 方法 | 观测结果 | 判断 |
|---|---|---|
| 完整数学解答的全参数 SFT | 干净 SFT 分支在 300 题上由约 78.0% 提升到 80.3%，checkpoint 平均带来额外收益 | 有效 |
| GSM8K 训练题聚焦继续训练 | 300 题 87.0%，完整集约 83.40% | 显著有效 |
| 独立 MetaMath LoRA | 300 题峰值约 80.7%，低于全参数聚焦训练 | 未采用 |
| 精确答案 GRPO | 最强早期导出完整集约 83.62% | 小幅优于聚焦 SFT，但仍低于后续蒸馏 |
| 已验答案 R1 推理蒸馏 | 完整集最高约 86.28%，晚期 checkpoint 优于早期 checkpoint | 主要增益来源之一 |
| checkpoint 插值 | 35% reasoning 权重时约 84.76%；更强插值在独立算术检查中出现空输出 | 中等比例可用，过强会退化 |
| 正确 rollout 的拒绝采样 SFT | 完整集约 80.29%–81.43% | 明显低于蒸馏分支，未采用 |
| 最终强化学习继续训练与温度选择 | Agent 内部最佳约 86.66%；温度 0.15 和 0.45 均约 85.75%，温度 0.3 最好 | 最终采用 |

## 最终选择

报告记录的最终候选为 refine_temperature/checkpoint-1，来源于 R1 推理蒸馏后的强化学习细化，并采用 temperature=0.3。Agent 内部完整评测为 86.66%，控制器复评为 86.13%；正式结果采用后者。

## 解释边界

报告中的 300 题开发评测、不同 token 上限以及完整评测不能混合比较。由于缺少该运行自己的控制器 base_full，摘要只报告正式 final_full，不推断统一口径的基础模型增量。

## 主要证据

- controller/final_full/metrics.json
- agent_home/task/TRAINING_REPORT.md
- agent_home/task/final_model/training_summary.json
- agent_home/task/experiments/final_selection.json
- records/agent_model_outputs.jsonl

