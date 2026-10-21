# 实验摘要：qwen17_gsm8k_5h_r04

## 基本信息

- 基础模型：Qwen/Qwen3-1.7B-Base
- 下游任务：GSM8K（1,319 题）
- Agent：gpt-6.1-sol，reasoning effort=high
- 时间预算：5 小时
- 状态：完整结束并完成控制器 final_full 复评

## 正式结果

控制器 final_full 为 1,101/1,319（83.47%）。该运行目录没有控制器 base_full/metrics.json，因此不能计算统一口径的正式净提升。Agent 报告中的未训练基础模型只在 100 或 150 题前缀上评测，与完整集不可直接比较。

## 方法与效果

| 方法 | 观测结果 | 判断 |
|---|---|---|
| 训练题教师解答 SFT 与 checkpoint 平均 | 多个完整集候选约 77.3%–82.5% | 建立了较强基础 |
| 增加 60,000 个 OpenMath 多样化训练题 | diverse 系列约 81.3%–82.5%，报告称其带来最大的阶段性增益 | 有效 |
| 本地生成并验答案的 RFT | rft_200/300/400 为 78.24%/80.06%/79.15% | 低于领先模型，未采用 |
| DPO | 早期 dpo_60 为 82.64%，继续到 dpo_120 降到 80.06% | 早期有效，过度训练明显退化 |
| MATH 训练集桥接课程 | math_bridge_200 为 82.94%，优于相邻 checkpoint | 有效，存在明确最佳训练步数 |
| R1 推理混合 | r1_mix_240 为 83.47%，高于 80/160 step；纯 R1 分支较弱 | 低剂量混合有效 |
| 数学 DPO 细化 | dpo_math_120 为 83.70%，优于 dpo_math_40/80 | 有效 |
| 参数平均 | equal_repeat 和 final_blend_equal 的 agent 内部单次完整评测为 84.00%，高于各父模型 | 最终采用 |
| 额外教师多样性继续训练 | novel2 系列约 81.7%–82.8% | 未超过领先模型 |

## 最终选择

最终模型为 checkpoints/final_blend_equal，即最佳偏好分支和推理混合分支的等权参数平均。Agent 报告的最佳单次完整评测为 84.00%，三次匹配评测均值为 83.67%；控制器复评为 83.47%，正式结果采用控制器值。

## 退化或失败现象

- DPO 从 60 step 继续到 120 step 后明显下降。
- 部分 tagged R1 模型会立即输出 EOS，未进入最终选择。
- 更强训练、更多新数据或更晚 checkpoint 并不必然更好。
- 已验证 SFT 虽提高保留训练验证集分数，却降低完整 benchmark 分数，因此被拒绝。

## 主要证据

- controller/final_full/metrics.json
- agent_home/task/REPORT.md
- agent_home/task/research_notes.md
- agent_home/task/experiments/
- records/agent_model_outputs.jsonl

