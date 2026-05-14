# V5 / OpenClaw 全工作流打通 Codex 交付报告

日期：2026-05-15
分支：codex/v5-openclaw-workflow-closure
角色：Codex 总工程承包方

## 结论

V5 学习链路已从“parser 直接写 models.json”改为：

OpenClaw / memory 输入 -> 多解析器 -> LearningEvent -> candidate_models -> candidates_pending.json -> CEO/用户确认 -> memory/v5/models.json -> memory/v5/model-changes.jsonl。

Codex 已验证 dry-run、apply、重复 apply 幂等、确认写入路径和回归脚本。CEO 仍保留最终实验、验证、审核和是否合并的决定权。

## 修改文件

- `memory_v4/behavior_learner.py`
  - 支持 `--date` 作为回溯锚点，配合 `--days` 读取多日 memory 文件。
  - 新增 LearningEvent 中间层，支持 required event types。
  - `--dry-run` 不写任何事实源。
  - `--apply` 只幂等写入 `memory_v4/candidates_pending.json`。
  - 报告新增 `parsing_summary.parsed_samples`、`learning_events`、`learning_event_details`、pending 写入统计。
- `memory_v4/confirm_candidate.py`
  - 确认路径改为写 `memory/v5/models.json`。
  - 通过 V5 engine 写入，自动更新 `models.md` 和 `model-changes.jsonl`。
- `test_v5_learning_regression.sh`
  - 回归脚本更新为 V5 新闭环语义。
- `ai-company/agent_dispatch_bridge.py`
  - 工作流状态改为 `created / dispatched / acked / running / completed / failed`。
  - `completed` 后等待 CEO 手动 `verified`。
- `ai-company/task_ledger.py`
  - task ledger 统计和终态支持 `completed / failed / verified`，保留旧 `idle / error` 兼容。

`ai-company` 是独立仓库/submodule，以上两项已单独提交到 `abao48274-alt/ai-company` 的 `codex/openclaw-workflow-states` 分支，不混入 `qianchen-memory` 主提交。

## 运行命令

```bash
python3 -m py_compile memory_v4/behavior_learner.py memory_v4/confirm_candidate.py
python3 -m py_compile ai-company/agent_dispatch_bridge.py ai-company/task_ledger.py
python3 memory_v4/behavior_learner.py --date 2026-05-14 --dry-run --output-report /tmp/v5_learning_report.json
python3 memory_v4/behavior_learner.py --date 2026-05-14 --apply --output-report /tmp/v5_apply_report.json
./test_v5_learning_regression.sh
python3 memory_v4/confirm_candidate.py '1确认'
```

确认测试已使用备份恢复，不保留测试写入污染。

## dry-run 报告摘要

- `status`: `dry_run_completed`
- `input_summary.files_read`: `memory/2026-05-14.md`, `memory/2026-05-13.md`, `memory/2026-05-12.md`
- `parsing_summary.parsed_samples`: 22
- `learning_events`: 25
- `candidate_details`: 2
- `output_summary.models_json_modified`: false
- `output_summary.model_changes_jsonl_modified`: false

## apply 报告摘要

- `status`: `candidates_pending_updated`
- `candidates_pending_added`: 2
- `candidates_pending_total`: 2
- `candidate` 必含字段：`evidence`, `source`, `type`, `confidence`
- `models_json_modified`: false
- `model_changes_jsonl_modified`: false
- 重复 apply：`candidates_pending_added = 0`，pending 文件哈希不变

## 回归样本覆盖

使用 `memory/2026-05-14.md` 提取到以下关键学习事件：

- 没有 ACK，不算派工
- 没有 worker，不算 Agent 可执行
- message_bus 只做状态副本，不做主执行通道
- 没有 Codex 原始调用证据不算执行完成
- PR 合并前必须终审
- 脚本运行成功不等于学习完成：由 PR 验证上下文推导为 validation_failed 类风险，不伪造样本文本证据

## 测试结果

- `py_compile`: 通过
- 指定 dry-run 验收：通过
- apply 只写 pending：通过
- 重复 apply 幂等：通过
- `learn_from_today()` 兼容：通过
- 异常输入状态：通过
- `test_v5_learning_regression.sh`: 5/5 通过
- 确认路径测试：pending 1 条确认后 `models.json` 34 -> 35，`model-changes.jsonl` 107 -> 108，随后已恢复

## 未验证项

- CEO 尚未在自己的最终验收环境中运行完整验收套件。
- OpenClaw 真实 agent dispatch 需要 CEO 用实际 task-ledger 任务触发；Codex 当前只验证了桥接代码语义和语法。
- PR 合并、main 更新、生产化 timer/service 重启均未执行。

## 风险项

- 当前工作区存在大量既有脏文件和运行态文件，本次提交只应包含上述相关文件。
- `memory/v5/models.json` 在本次工作前已经有未提交改动；Codex 本轮未把 `--apply` 写入 models.json。
- `ai-company` 是独立 Git 仓库或 submodule，已单独推送分支；是否合并和是否更新主仓库 submodule 指针由 CEO 决定。

## 回滚方案

- 回滚 V5 学习器：撤销 `memory_v4/behavior_learner.py` 和 `memory_v4/confirm_candidate.py`。
- 回滚回归脚本：撤销 `test_v5_learning_regression.sh`。
- 回滚 OpenClaw 状态流：撤销 `ai-company/agent_dispatch_bridge.py` 和 `ai-company/task_ledger.py`。
- 若运行 apply 产生 pending 候选，可将 `memory_v4/candidates_pending.json` 恢复为运行前备份或清空为 `[]`。
- 不需要回滚 `models.json`，因为本实现的 `--apply` 不直接写该文件。

## CEO 验收建议

```bash
python3 memory_v4/behavior_learner.py --date 2026-05-14 --dry-run --output-report /tmp/v5_learning_report.json
python3 memory_v4/behavior_learner.py --date 2026-05-14 --apply --output-report /tmp/v5_apply_report.json
./test_v5_learning_regression.sh
```

CEO 验收重点：确认报告字段、pending 候选质量、重复 apply 幂等、以及是否批准将 pending 候选写入 V5 models。
