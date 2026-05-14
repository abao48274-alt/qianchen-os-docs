# 三方共享仓库切换记录

时间：2026-05-15  
来源：Codex  
任务：SHARED-REPOSITORY-SWITCH-20260515  
对象：CEO / ChatGPT / Codex / OpenClaw

---

## 1. 结论

三方共享仓库已切换为 GitHub 私有仓库：

```text
abao48274-alt/qian-chen
```

本地路径：

```text
/home/administrator/.openclaw/workspace/qian-chen
```

remote：

```text
https://github.com/abao48274-alt/qian-chen.git
```

旧共享仓库暂时不用。后续涉及 ChatGPT、Codex、OpenClaw 三方协作的代码和项目事项，默认以 `qian-chen` 为共享事实源。

---

## 2. 当前可访问状态

用户确认：

- Codex：已创建环境，可访问。
- ChatGPT：已可读取仓库。
- OpenClaw：已可正常读取仓库。

Codex 本地核验：

- `qian-chen` 目录存在。
- git remote 指向 `https://github.com/abao48274-alt/qian-chen.git`。
- 当前分支为 `main`，跟踪 `origin/main`。
- 仓库内已有 `README.md` 与 `AGENTS.md`。

---

## 3. 协作规则

`qian-chen/AGENTS.md` 已定义：

- 不直接推送 main，除非用户明确要求紧急直改。
- 使用独立分支：
  - `chatgpt/<task-name>`
  - `codex/<task-name>`
  - `openclaw/<task-name>`
- 优先通过 PR 做可审查变更。
- 不提交 secrets、API keys、tokens、cookies、`.env`。
- 编辑前先读取仓库最新文件。
- 小范围、聚焦提交。

---

## 4. 对 CEO 的要求

CEO 需要更新默认协作认知：

1. 以后三方共享代码仓库是 `abao48274-alt/qian-chen`。
2. 旧共享仓库暂时不用，不再作为默认协作入口。
3. OpenClaw 派工涉及共享代码时，优先检查 `/home/administrator/.openclaw/workspace/qian-chen`。
4. 子 Agent 执行前必须读取 `qian-chen/AGENTS.md`。
5. 需要写入 memory，避免下次仍引用旧仓库。

---

## 5. Codex 额度说明

当前 Codex 对话仍可继续，但 Codex / Codex Runner 的外部执行额度无法由本报告判断是否恢复。

规则：

- 不为确认额度而消耗高成本调用。
- 只有在用户明确要求或关键任务必须使用 Codex 时，才进行真实 Codex 调用。
- 若再次遇到 usage limit，应记录到报告和 message_bus，并进入 cooldown。

---

## 6. 短消息

```text
Shared repo switched: qianchen-os-docs/reports/2026-05-15-shared-repository-switch-codex.md. Default shared repo is abao48274-alt/qian-chen. Write memory and use qian-chen going forward.
```
