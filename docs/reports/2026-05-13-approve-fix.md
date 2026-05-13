# Approve API 修复报告

**日期**: 2026-05-13 11:01 GMT+8
**修复人**: 千尘 CEO（亲自执行，Codex 未完成）
**范围**: P0 only — 修复 approve 500 错误

---

## 修复内容

| 项目 | 修复前 | 修复后 |
|------|--------|--------|
| `/api/proposals/approve` | HTTP 500 (TASKS_FILE undefined) | HTTP 200 ✅ |
| `/api/bridge/approved-to-board` | HTTP 500 (级联失败) | HTTP 200 ✅ |
| 任务创建 | 0 个 | 正常创建 ✅ |
| task-ledger.json | 无会议任务 | 有会议任务 ✅ |

## 技术变更

1. **删除所有 `TASKS_FILE` 引用** — 该变量从未定义
2. **改用 `task_ledger.create_task()`** — 通过模块 API 写入任务
3. **`_get_meeting_context()` 用 `_list_tasks()`** — 读取真实任务数据
4. **修复状态字段** — task_ledger 用 `state` 而非 `status`
5. **bridge 改为时间过滤** — 近 5 分钟创建的任务视为会议任务

## 验收结果

```
1. Approve: HTTP 200, 创建 2 个任务 ✅
2. task-ledger.json: 会议任务已写入 ✅  
3. Bridge: HTTP 200, 返回 7 个任务 ✅
```

## 未做（P1/P2/P3 后续）

- ❌ 不自动派工
- ❌ 不触发 Codex
- ❌ 不接 Telegram
- ❌ 不做 LLM 发言
- ❌ 不改会议 UI

## 经验教训

- `task_ledger` 模块的 `create_task()` 参数是 `title/input_desc/project_id/priority/assignee`，不是 `task_id`（自动生成）
- `_list_tasks()` 返回 list，task-ledger.json 文件格式是 `{"tasks": [...]}`
- task_ledger 用 `state` 字段，不是 `status`
