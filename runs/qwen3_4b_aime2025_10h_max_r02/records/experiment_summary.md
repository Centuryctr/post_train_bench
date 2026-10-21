# 实验摘要：qwen3_4b_aime2025_10h_max_r02

## 基本信息

- 基础模型：Qwen/Qwen3-4B-Base
- 下游任务：AIME 2025（30 题）
- Agent：gpt-6.1-sol，reasoning effort=max
- 时间预算：10 小时
- 状态：完整结束并完成控制器 final_full 复评

## 正式结果

控制器统一评测中，基础模型为 2/30（6.67%），最终模型为 8/30（26.67%），净提升 6 题、20.00 个百分点。Agent 在训练阶段对所选候选曾测得 10/30（33.33%），但控制器独立复评为 8/30，因此正式结果采用 8/30。

## 方法与效果

| 方法 | 观测结果 | 判断 |
|---|---|---|
| 2025 年考试前数学推理数据上的 LoRA | step 50 为 2/30，step 100 降到 0/30；输出经常过长且不能结束 | 收益不稳定，后期退化 |
| 推理、聊天和停止 marker 的定向校准 | LoRA 分支由约 2/30 提高到 5/30 | 明显改善输出终止 |
| 全参数训练加 marker 校准 | 8,192-token 内部评测达到 7/30 | 进一步提升 |
| 更宽泛的长程数学继续训练 | 多个 checkpoint 在 6/30 左右平台，未超过已有最佳 | 无进一步收益，停止 |
| 历史竞赛题聚焦训练加 greedy 解码 | 内部评测达到 10/30 | 形成最强父模型 |
| 历史问题本地 rollout 与普通 GRPO | 首个 GRPO checkpoint 为 6/30，低于父模型；第二轮出现少量 token 上的大 KL 发散 | 变差或不稳定 |
| 修正后的 Dr.GRPO | 全八卡比较为 8/30 | 比部分监督候选好，但低于最终 verified-SFT |
| 只使用验答案正确 rollout 的 SFT | 全八卡内部比较为 10/30 | 最终采用 |
| repetition penalty 与其他解码设置 | 两个已完成变体均约 8/30；最后一个试验因截止时间未完成而排除 | 未超过最终模型 |

## 最终选择

最终提交是从聚焦数学父模型出发、使用本地生成且经历史答案验证的正确解答继续 SFT 的候选。它在 agent 内部全八卡评测中为 10/30；控制器复评为 8/30。

## 解释边界

30 题评测方差较大，且训练过程多次使用 AIME 2025 聚合分数选择 checkpoint 和解码参数。内部 10/30 与控制器 8/30 的差异说明单次分数不稳定；方法级结果应按方向性证据解读。

## 主要证据

- controller/base_full/metrics.json
- controller/final_full/metrics.json
- records/agent_model_outputs.jsonl
- records/agent_system_outputs.jsonl
- agent_home/task/final_model/ 中的模型来源与校验文件

