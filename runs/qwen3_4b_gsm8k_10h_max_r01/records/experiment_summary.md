# 实验摘要：qwen3_4b_gsm8k_10h_max_r01

## 基本信息

- 基础模型：Qwen/Qwen3-4B-Base
- 下游任务：GSM8K（1,319 题）
- Agent：gpt-6.1-sol，reasoning effort=max
- 时间预算：10 小时
- 状态：完整结束并完成控制器 final_full 复评

## 正式结果

控制器统一评测中，基础模型为 560/1,319（42.46%），最终模型为 1,219/1,319（92.42%），净提升 659 题、49.96 个百分点。

Agent 另做了一组严格 greedy 的完整集控制：未训练基础模型为 80.52%，所选模型为 92.57%，同口径提升 12.05 个百分点。该组生成设置与控制器基线不同，因此两套增量分别保留，不能混合。

## 方法与效果

| 方法 | 同口径完整集结果 | 判断 |
|---|---:|---|
| native-EOS 监督训练 | 90.45% | 主要基础增益，正确停止符非常重要 |
| 十样例 prompt 对齐继续训练 | 91.36% | 继续提升 |
| 小规模 R1 推理更新 | 91.58% | 小幅提升 |
| GSM8K 训练题 DPO | 91.89% | 有效，但未达到最终最佳 |
| 原训练题 LoRA GRPO | 91.81% | 有效但略低于 DPO |
| 推理轨迹蒸馏 | 92.19% | 进一步提升 |
| 扩展课程全参数 GRPO | 92.57% | 最佳结果，选入最终模型 |
| 已验证 self-checking SFT | 92.27% | 接近最佳但未超过 |
| 使用剩余官方训练题的最终拟合 | 92.57% | 与最佳持平，没有明确额外增益 |
| 从最佳父模型继续偏好优化 | 92.04% | 变差 |
| 偏好/RL 权重平均 | 最好约 91.81% | 变差 |
| 选择性参数平均 | 最好约 92.27% | 未超过最佳 |

## 最终选择

最终模型来自 checkpoints/grpo_full/merged_checkpoint-120，即扩展课程上的全参数 GRPO checkpoint。Agent 内部同口径结果为 1,221/1,319（92.57%）；控制器最终复评为 1,219/1,319（92.42%）。

## 关键负面发现

- 激活过长推理有时会破坏答案结尾和停止行为。
- 在最佳模型上继续做偏好训练、模型平均或权重算术通常没有继续提升。
- native EOS 和明确的生成停止设置是性能与可用性的关键组成部分。

## 解释边界

Agent 报告中的 greedy control 与控制器默认评测属于不同生成设置。正式排行榜结果采用控制器 base_full/final_full；方法级比较采用报告中明确声明为相同完整集、相同 greedy 设置的实验表。

## 主要证据

- controller/base_full/metrics.json
- controller/final_full/metrics.json
- agent_home/task/RESULTS.md
- agent_home/task/artifacts/controlled_full_results.csv
- agent_home/task/artifacts/best_selection.json
- records/agent_model_outputs.jsonl

