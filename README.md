1. `agent_model_outputs.jsonl`
agent 自己产生的文本，包含两类：
`agent_message`：Codex 明确输出的阶段总结、进度说明、最终回答等。
`reasoning_summary`：Codex 暴露出来的推理摘要。
结构如下：
```python
{
  "schema_version": 1,
  "run_id": "qwen3_4b_aime2025_10h_max_r02",
  "sequence": 42,
  "source_line": 42,
  "source": "codex",
  "kind": "reasoning_summary",
  "item_id": "item_12",
  "text": "正在比较不同训练数据和训练方案……"
}
```
字段解释：`run_id`：所属实验。
`sequence` / `source_line`：对应原始 agent.jsonl 的行号，可用于回溯。
`kind`：`agent_message` 或 `reasoning_summary`。
`item_id`：Codex 事件 ID。
`text`：Agent 输出的文本。

2. `agent_system_outputs.jsonl`
agent 调用工具和 Codex 运行时产生的记录，包含几类：
`command_execution`：Agent 执行的命令、退出码和命令输出。
`file_change`：Agent 发起的文件修改及修改状态。
`web_search`：Agent 发起的网络搜索、动作和搜索结果。
`runtime_message`：原始 `agent.jsonl` 中无法解析为 JSON 的 stderr 或运行时文本。
`warning` / `error`：Codex 自身报告的警告或错误事件。
`unfinished_item`：已经开始但在日志结束前没有完成的工具调用。
结构如下：
```python
{
  "schema_version": 1,
  "run_id": "qwen3_4b_aime2025_10h_max_r02",
  "sequence": 6,
  "source_line": 6,
  "source": "codex_tool",
  "kind": "command_execution",
  "item_id": "item_1",
  "started_source_line": 5,
  "command": "/bin/bash -lc 'pwd && ls -la'",
  "status": "completed",
  "exit_code": 0,
  "output": "/root/emhua/qrzeng/post_train_bench/..."
}
```
字段解释：`run_id`：所属实验。
`sequence` / `source_line`：对应原始 `agent.jsonl` 的行号，可用于回溯。
`source`：记录来源，例如 `codex_tool`、`codex_runtime` 或 `codex_stderr_or_unstructured`。
`kind`：工具或运行时事件类型。
`item_id` / `started_source_line`：工具事件 ID 及其开始事件所在行。
`command` / `status` / `exit_code` / `output`：命令内容、执行状态、退出码和聚合输出；历史 Codex 命令输出可能混合 stdout 与 stderr。
`query` / `action` / `results`：仅网络搜索事件具有的搜索字段。
`changes`：仅文件修改事件具有的修改明细。
`payload`：其他事件无法归入固定字段的原始内容。

注意：这里的 `system` 指 Codex 工具层，不是 Linux 系统输出。Linux、GPU、评测进程和独立 stderr 记录在 `linux_system_outputs.jsonl`。

3. `eval_model_outputs.jsonl`
基础模型和最终模型在下游评测中产生的逐样本输出，正式评测通常包含两类：
`base_full`：Agent 训练开始前的基础模型评测。
`final_full`：Agent 训练结束后的最终模型评测。
结构如下：
```python
{
  "schema_version": 1,
  "run_id": "qwen3_4b_aime2025_10h_max_r02",
  "task": "aime2025",
  "evaluation": "final_full",
  "inspect_log": "final_full/inspect_logs/example.json",
  "sample_index": 0,
  "sample_id": "0",
  "uuid": "example-uuid",
  "epoch": 1,
  "input": "<benchmark input>",
  "model_output": "<model completion>",
  "model": "/root/emhua/qrzeng/post_train_bench/runs/.../final_model",
  "usage": {"input_tokens": 131, "output_tokens": 62, "total_tokens": 193},
  "target": "<reference answer>",
  "scores": {"aime_scorer": {"value": "C"}},
  "metadata": {},
  "error": null,
  "total_time": 5.398,
  "working_time": 1.327
}
```
字段解释：`run_id` / `task`：所属实验和下游任务。
`evaluation`：评测阶段，通常为 `base_full` 或 `final_full`。
`inspect_log` / `sample_index` / `sample_id` / `uuid` / `epoch`：原始 Inspect AI 日志和样本定位信息。
`input`：下游任务输入。
`model_output` / `model`：模型生成的文本和产生该文本的模型路径。
`usage`：该样本的输入、输出和总 token 数。
`target` / `scores`：参考答案和评分器结果。
`metadata` / `error`：Inspect AI 保存的样本元数据和错误信息。
`total_time` / `working_time`：样本的总耗时和实际工作耗时。

注意：该文件可能包含测试题和标准答案，只能用于评测分析，不能重新提供给 Agent 训练。

4. `linux_system_outputs.jsonl`
本次实验范围内的 Linux、进程和运行时记录，包含几类：
`process_start` / `process_exit`：Codex 或评测进程的启动信息和退出码。
`stream_line`：Codex 或评测进程分离采集的 stdout、stderr 文本行。
`resource_snapshot`：默认每 30 秒采样的 CPU load、内存、实验磁盘、8 张 GPU 和实验进程树。
`monitor_start` / `monitor_stop`：资源监控器的开始和结束事件。
`log_line`：从历史实验已有日志中保守回填的 warning/error 行。
`collector_parse_error`：结构化系统事件无法解析时保留的原始文本和解析错误。
结构如下：
```python
{
  "schema_version": 1,
  "run_id": "qwen3_4b_aime2025_10h_max_r02",
  "sequence": 12,
  "source": "linux_system",
  "source_path": "controller/final_full/system_stream_events.jsonl",
  "source_line": 12,
  "timestamp": "2026-10-20T03:04:15.123456+00:00",
  "event_type": "stream_line",
  "stage": "final_full",
  "component": "evaluator",
  "stream": "stderr",
  "severity": "error",
  "message": "CUDA out of memory"
}
```
字段解释：`run_id` / `sequence`：所属实验和该文件内的记录顺序。
`source_path` / `source_line`：原始结构化事件或历史日志的位置，可用于回溯。
`timestamp`：事件发生时间；历史日志没有可靠时间时为 `null`，不会伪造时间。
`event_type`：系统事件类型。
`stage` / `component`：事件所属阶段和产生事件的组件。
`stream`：`stdout`、`stderr` 或历史合并流 `combined`。
`severity`：根据事件或文本归类的 `info`、`warning`、`error`。
`message`：去除 ANSI 控制符后的日志文本；过长文本会标记 `message_truncated`。
`payload`：资源快照、进程信息、退出码等结构化附加内容。

注意：采集范围仅限本次实验目录、实验进程树和分配的 GPU，不读取共享服务器的全局 `journalctl` 或 `dmesg`。历史实验若已经通过 `2>&1` 合并输出，只能标记为 `combined`，不能重新恢复 stdout/stderr 边界。

5. `manifest.json`
上述四份 JSONL 的生成清单，是一个普通 JSON 文件，包含几类信息：
`generator`：规范化导出程序及其版本。
`semantics`：四份输出文件各自的语义说明。
`sources`：原始 `agent.jsonl` 和 Inspect AI 日志的路径、大小及 SHA-256。
`agent_trace`：Agent 日志的有效行、无效行、事件类型、token 用量和未完成事件统计。
`linux_system`：结构化系统来源、历史日志来源、错误、事件类型和严重级别统计。
`outputs`：四份 JSONL 的文件名、记录数、大小及 SHA-256。
`privacy`：文件是否经过脱敏以及分享提醒。
结构如下：
```python
{
  "schema_version": 1,
  "generator": {"name": "export_normalized_records.py", "version": "1.1.0"},
  "generated_at": "2026-10-20T09:00:30.693955+00:00",
  "run_id": "qwen3_4b_aime2025_10h_max_r02",
  "task": "aime2025",
  "run_root": "/root/emhua/qrzeng/post_train_bench/runs/qwen3_4b_aime2025_10h_max_r02",
  "semantics": {"agent_model_outputs": "Completed Codex agent messages and exposed reasoning summaries."},
  "sources": {"agent_log": {"path": "agent_home/task/agent.jsonl", "bytes": 2085898, "sha256": "..."}, "inspect_logs": [], "inspect_log_errors": []},
  "agent_trace": {"exists": true, "valid_json_lines": 1992, "invalid_json_lines": 12, "unfinished_item_ids": []},
  "linux_system": {"records": 688, "structured_sources": [], "legacy_sources": [], "source_errors": []},
  "outputs": {
    "agent_model": {"path": "agent_model_outputs.jsonl", "records": 121, "bytes": 60352, "sha256": "..."},
    "agent_system": {"path": "agent_system_outputs.jsonl", "records": 946, "bytes": 1800000, "sha256": "..."},
    "eval_model": {"path": "eval_model_outputs.jsonl", "records": 60, "bytes": 3900000, "sha256": "..."},
    "linux_system": {"path": "linux_system_outputs.jsonl", "records": 688, "bytes": 403000, "sha256": "..."}
  },
  "privacy": {"sanitized": false, "note": "Derived records can contain sensitive text. Review before sharing."}
}
```
字段解释：`schema_version`：规范化记录格式版本。
`generator` / `generated_at`：生成程序、程序版本和清单生成时间。
`run_id` / `task` / `run_root`：所属实验、任务和实验根目录。
`semantics`：四份派生文件的准确语义。
`sources`：所有原始输入及其校验信息；无法读取的 Inspect 日志记录在 `inspect_log_errors`。
`agent_trace`：Agent 轨迹的解析统计、事件统计、线程信息、token 用量和未完成调用。
`linux_system`：系统记录总数、来源、解析错误以及按事件和严重级别汇总的统计。
`outputs`：每份派生 JSONL 的路径、记录数、字节数和 SHA-256，可用于完整性核验。
`privacy`：脱敏状态和分享前检查提示。

注意：`manifest.json` 不保存新的模型输出或系统输出，只描述记录如何生成并提供统计与校验信息。当前规范化文件未经脱敏，分享前必须检查。

手动重建某个历史 run：

```bash
cd /root/emhua/qrzeng/post_train_bench/local_harness
EXPERIMENT_ENV=/root/emhua/qrzeng/post_train_bench/local_harness/experiment.env \
  bash export_records.sh RUN_ID
```

该命令只替换 `controller/records/` 下的派生文件，不改动原始日志、模型或评测结果。
