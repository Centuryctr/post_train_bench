# 实验摘要：qwen17_gpqamain_10h_max_r03

## 基本信息

- 基础模型：Qwen/Qwen3-1.7B-Base
- 下游任务：GPQA Main（448 题）
- Agent：gpt-6.1-sol，reasoning effort=max
- 时间预算：10 小时
- 状态：完整结束并完成控制器 final_full 复评

## 正式结果

控制器统一评测中，基础模型为 62/448（13.84%），最终模型为 165/448（36.83%），净提升 103 题、22.99 个百分点。

## 方法与效果

| 方法 | 观测结果 | 判断 |
|---|---|---|
| 完整科学推理的全参数 SFT | 明显改善指令遵循；长自由回答推理向选择题迁移不如直接标签稳定 | 有效，但需要目标格式训练 |
| 四选项均衡标签 SFT | 300 step 完整评测约 35.7%，后续 checkpoint 下降 | 前期提升，继续训练导致退化 |
| 本地教师概率蒸馏 | 单模型约 35.5%；与直接监督 checkpoint 平均后曾出现 37.05% 的单次完整评测 | 有效，但单次结果受选项重排影响 |
| 新问题扩充 | 完整评测约 33.0%–34.2% | 低于领先候选，未替换 |
| 困难干扰项 SFT | 最好约 34.38% | 有一定能力，但低于领先候选 |
| 可验证答案 GRPO | 分类器型 GRPO 配合确定性 top-k=1 解码，在 30 次独立选项重排上的平均值约 35.62% | 稳定性较好，成为最终选择 |
| 权重平均与多分支组合 | 个别单次分数较高，但重复评测均值未稳定超过最终 GRPO 候选 | 未采用 |

## 最终选择

最终模型对应 grpo_classifier200_greedy 路线。Agent 没有按单次最高分直接选择，而是对同一 448 题进行 30 次独立答案选项重排，并按平均准确率选择；控制器随后复评得到 36.83%。

## 退化或未采用路线

- 四选项 SFT 的晚期 checkpoint 低于 300 step。
- fresh-question expansion 和 hard-distractor SFT 未超过领先模型。
- 历史上 37.05% 的单次权重平均结果没有在重复重排均值上形成稳定优势。
- 长推理训练的迁移效果不如直接选择题监督稳定。

## 解释边界

GPQA 评测会重排答案选项，单次结果存在明显波动。多次评测仍然是同一批 448 题，不等于增加独立测试样本；而且 GPQA 分数参与了模型选择，因此应把方法比较视为开发过程证据。

## 主要证据

- controller/base_full/metrics.json
- controller/final_full/metrics.json
- agent_home/task/research/experiment_summary.json
- agent_home/task/research/final_report.json
- records/agent_model_outputs.jsonl
- records/agent_system_outputs.jsonl

