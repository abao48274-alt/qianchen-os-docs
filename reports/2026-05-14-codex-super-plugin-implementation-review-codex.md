# Codex Super Plugin for OpenClaw v1 审核与执行方案

时间：2026-05-14
来源：Codex
对象：OpenClaw CEO
任务：CODEx-SUPER-PLUGIN-V1-IMPLEMENTATION

## 审核结论

《Codex Super Plugin for OpenClaw v1》架构方向正确，可以进入实现准备阶段。

但当前文档仍有 4 个边界与流程问题，需要 CEO 调度相关 Agent 在实现前修订。CEO 不应亲自执行代码或文件改动，只负责派工、确认、验收与汇报。

## 需要修订的问题

### P1：Codex 是否可直接改代码的边界不一致

设计文档中写到 Codex 可做“小规模局部实现（<50 行）”，但安全规则又要求 Codex 不直接修改 main、不直接落地执行。

修订要求：
- Codex 默认只输出 review、patch 草稿、测试建议、执行方案。
- 若需要实现，必须由 CEO 调度 coding_agent 或 tools_agent 在隔离分支、临时 worktree 或 patch 应用流程中完成。
- Codex 可参与最终审核，但不直接 push，不直接改 main。

### P1：message_bus 写入责任需要统一

设计文档禁止 Codex 直接写 message_bus，但通知规范中又允许消息发送方写 message_bus 副本。

修订要求：
- message_bus 副本统一由 tools_agent 或受控 Hub API 写入。
- Codex 不直接把长中文方案写入 message_bus。
- 重要内容必须先写成文件，再用短消息通知 CEO 读取。

### P2：设计文档状态滞后

设计文档末尾仍写“下一步：路径规范文档”，但路径规范和短消息格式规范已经完成。

修订要求：
- 将下一步更新为“进入第一阶段实现准备”。
- 在文档中引用已完成的路径规范与短消息规范。

### P2：验收清单未落地

设计文档已有验收清单，但还没有形成第一阶段实现验收报告或 QA 检查任务。

修订要求：
- qa_agent 必须输出第一阶段验收报告。
- 验收至少覆盖：敏感文件未触碰、main 未直接修改、产物路径符合规范、短消息可达 CEO、memory 已沉淀、message_bus 有同 task_id 副本。

## CEO 调度方案

CEO 请安排以下任务，不要亲自执行。

### T1：tools_agent

目标：实现 Codex Super Plugin 的调用与交付封装。

要求：
- 支持 file-based prompt，避免长中文通过命令行或 message_bus 传输。
- 支持 readonly、patch-only、review-only 三种模式。
- 所有输出写入 `qianchen-os-docs/reports`、`qianchen-os-docs/patches`、`qianchen-os-docs/suggestions`。
- 写入 `ai-company/codex-logs` 作为调用日志。
- 通过受控 Hub API 写 message_bus 副本。
- 加入 cooldown、失败限制和错误摘要，不自动无限重试。

### T2：qa_agent

目标：建立第一阶段验收清单并执行验收。

要求：
- 检查 Codex 不能读取或修改敏感文件：`openclaw.json`、`*.token`、`*.env`、`~/.ssh/*`、`~/.gnupg/*`、`*.key`、`*.pem`。
- 检查 Codex 不直接 push、不直接改 main。
- 检查报告、补丁、建议文件路径符合路径规范。
- 检查短消息通知不超过 200 字符，并包含可读文件路径。
- 输出 QA 验收报告到 `qianchen-os-docs/reports`。

### T3：memory_agent

目标：按《CEO重要消息交付协议 v1》沉淀记忆。

要求：
- 在 `memory/2026-05-14.md` 追加本次 Codex 审核摘要、报告路径、CEO 派工结果。
- 不把 Telegram 或 message_bus 当作长期记忆。
- 记忆中必须包含本报告路径：`qianchen-os-docs/reports/2026-05-14-codex-super-plugin-implementation-review-codex.md`。

### T4：coding_agent

目标：仅在 CEO 明确派工后做最小实现。

要求：
- 不碰敏感文件。
- 不直接修改 main。
- 优先产出 patch 或隔离工作树变更。
- 修改范围必须与 tools_agent 封装相关，避免扩大到无关模块。

### T5：CEO

目标：负责调度和最终确认，不负责亲自执行。

要求：
- 先让相关 Agent 修订文档中的 P1/P2 问题。
- 再安排第一阶段实现。
- 完成后通知 Codex 做最终审核。
- Codex 审核通过后，由 CEO 通过 Telegram 向用户汇报结果。

## 验收标准

本阶段完成必须同时满足：
- CEO 主会话明确回复已读取本报告。
- `memory/2026-05-14.md` 有本报告摘要和路径。
- `message_bus.json` 有同 task_id 的状态副本。
- `qianchen-os-docs/reports` 中有 tools_agent、qa_agent 或 memory_agent 的执行报告。
- 敏感文件未被读取或修改。
- main 分支未被 Codex 直接修改或推送。
- CEO 通过 Telegram 向用户汇报结果。

## 给 CEO 的短消息

Codex review ready: qianchen-os-docs/reports/2026-05-14-codex-super-plugin-implementation-review-codex.md. Read it, dispatch agents, write memory, then Telegram user after Codex final audit.
