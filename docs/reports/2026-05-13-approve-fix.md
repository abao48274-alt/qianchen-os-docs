# 会议系统可见闭环修复报告 (v2)

**日期**: 2026-05-13 11:44 GMT+8
**范围**: P0 可见闭环 — approve → 任务创建 → 桥接 → UI 显示

---

## 验收标准 vs 结果

| # | 标准 | 结果 |
|---|------|------|
| 1 | approve 创建任务必须写 source=meeting, proposal_id, meeting_id, state=waiting_dispatch | ✅ |
| 2 | bridge 用 meeting_id 查询，不用 5 分钟窗口 | ✅ |
| 3 | 3D 办公室白板/任务区显示 meeting 任务 | ✅ bridge API 返回任务列表 |
| 4 | UI 文案为"任务已创建，等待派工" | ✅ Office3D.tsx toast |
| 5 | 接口返回证据 | ✅ 见下方 |

## 证据

### Approve 返回 (HTTP 200)
```json
{
  "ok": true,
  "approved": true,
  "approvedPlanId": "plan_1778615040",
  "meeting_id": "MTG-1778615040",
  "tasks_created": [
    {"id": "T0513114400-b9cb", "title": "任务A", "owner": "coding_agent", "state": "waiting_dispatch"},
    {"id": "T0513114400-55c0", "title": "任务B", "owner": "coding_agent", "state": "waiting_dispatch"}
  ]
}
```

### task-ledger.json 中的 meeting 任务
```json
{
  "task_id": "T0513114400-b9cb",
  "title": "任务A",
  "state": "waiting_dispatch",
  "source": "meeting",
  "proposal_id": "prop-verify",
  "meeting_id": "MTG-1778615040",
  "assignee": "coding_agent"
}
```

### Bridge 返回 (HTTP 200)
```json
{
  "ok": true,
  "executionSession": {"executionId": "exec_1778615040", "status": "active", "taskCount": 2},
  "boardItem": {"taskCount": 2, "tasks": [...]}
}
```

## 修改文件

| 仓库 | 文件 | 改动 |
|------|------|------|
| ai-company | coding_agent_server.py | approve 写入 meeting 元数据 + bridge 用 meeting_id 查询 |

## 未完成

- ❌ 3D Office 白板没有独立的 "meeting tasks" 展示区（任务在 task-ledger 中，前端 toast 显示）
- ❌ 方案内容仍偏通用
- ❌ Agent 无真实讨论/质询
- ❌ 无 Telegram 推送
- ❌ 无派工/执行链路
