# 实验摘要：qwen3_4b_gpqamain_10h_max_r01

## 基本信息

- 基础模型：Qwen/Qwen3-4B-Base
- 下游任务：GPQA Main（448 题）
- Agent：gpt-6.1-sol，reasoning effort=max
- 时间预算：10 小时
- 状态：完整结束并完成控制器 final_full 复评

## 正式结果

控制器统一评测中，基础模型为 62/448（13.84%），最终模型为 152/448（33.93%），净提升 90 题、20.09 个百分点。

## 方法与效果

最终模型的主要训练链为：完整科学 SFT → 紧凑科学继续训练 → 科学偏好/GRPO → 本地参考约束蒸馏 LoRA → 数学 GRPO 热身 → 科学全参数 GRPO → top-k=1 确定性解码。

| 方法 | 可比内部结果 | 判断 |
|---|---|---|
| 完整科学推理 SFT | science_full/checkpoint-100 在完整 448 题、4,096 token 下约 29.46% | 建立基础能力，但距离最终模型仍有差距 |
| 紧凑科学继续训练 | 100 题筛选一度达到 41%，但该结果不能与完整集直接比较 | 有潜力，需完整评测确认 |
| 科学 LoRA GRPO | grpo_efficient/checkpoint-300 的 16,000-token 三次重排均值约 35.12% | 明显有效 |
| 本地参考约束的紧凑蒸馏 LoRA | condensed_lora/checkpoint-50 三次均值约 35.04% | 有效且接近领先模型 |
| 数学 GRPO 后再做科学 GRPO | 最终 science checkpoint 配合确定性解码形成最强重复均值 | 组合有效 |
| 教师 LoRA | 最好三次均值约 34.38%，晚期约 33.41% | 有提升但低于最终候选，晚期退化 |
| 拒绝采样 LoRA | 三次均值约 33.71%，且不同重排波动较大 | 未超过领先模型 |
| 晚期额外 science GRPO | 约 31.92%–32.37% | 变差 |
| 权重平均/model soup | qwen-branch-soup 约 34.38%；最终双模型 soup 在未完成结果的理论上界已低于 incumbent 时被提前淘汰 | 未带来提升 |

## 最终选择

最终提交权重为 checkpoints/greedy_variants/greedy_bd9629d41a，对应 math_science_grpo_full/checkpoint-50 的确定性 top-k=1 解码版本。Agent 使用三次完整 448 题选项重排均值选择，内部均值为 35.57%；控制器复评为 33.93%，正式结果采用后者。

## 解释边界

4,096-token 筛选、16,000-token 确认、100 题 pilot 和完整 448 题结果不可混合排序。重复选项重排仍使用同一批题，而且 GPQA 参与了模型选择，因此不属于完全未触碰的最终测试集。

## 主要证据

- controller/base_full/metrics.json
- controller/final_full/metrics.json
- agent_home/task/training_report.json
- agent_home/task/final_model/evaluation_summary.json
- agent_home/task/logs/replicated_summary.json
- records/agent_model_outputs.jsonl

