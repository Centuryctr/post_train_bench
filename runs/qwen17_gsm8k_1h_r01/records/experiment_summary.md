# 实验摘要：qwen17_gsm8k_1h_r01

## 基本信息

- 基础模型：Qwen/Qwen3-1.7B-Base
- 下游任务：GSM8K（1,319 题）
- Agent：gpt-6.1-sol，reasoning effort=high
- 时间预算：1 小时
- 状态：完整结束并完成控制器 final_full 复评

## 正式结果

控制器统一评测中，基础模型为 154/1,319（11.68%），最终模型为 923/1,319（69.98%），净提升 769 题、58.30 个百分点。

## 方法与效果

本实验主要采用全参数监督微调，没有引入强化学习。训练数据由 GSM8K train、严格映射回 GSM8K 训练题的 OpenMathInstruct-2 替代解答，以及训练集衍生的合成题组成。

| 方法 | 观测结果 | 判断 |
|---|---|---|
| 仅官方 GSM8K 解答的第一阶段 SFT | 100 题上从 epoch 1 的 31% 提升到 epoch 2/3 的 57% | 明显有效，后期趋于饱和 |
| 加入训练集衍生多样化问题的混合 SFT | 100 题约 62%–70%，优于只用官方解答 | 主要增益来源 |
| 低学习率细化 | 100 题为 67%，但在更可靠的 300 题比较中达到 71.3%，高于 mixture epoch1 的 69.0% | 小幅提升，最终采用 |
| 中间 checkpoint 选择 | mix-step250 为 62%，明显低于 mix-step500/epoch1 | 训练不足会变差 |

## 最终选择

final_model/selection.json 指向 checkpoints/refine-epoch1。它在 300 题开发比较中为 71.3%，是已比较候选中的最高值；控制器随后在完整 1,319 题上复评为 69.98%。

## 解释边界

100 题、300 题和完整集的分数不能直接互相计算增量。正式净提升只使用控制器 base_full 和 final_full；方法级结论来自报告中相同规模、相同生成设置的候选比较。

## 主要证据

- controller/base_full/metrics.json
- controller/final_full/metrics.json
- agent_home/task/REPORT.md
- agent_home/task/final_model/selection.json
- agent_home/task/logs/experiment_results.json
- records/agent_model_outputs.jsonl

