# 会议系统 P0 修复报告

**日期**: 2026-05-13 11:20 GMT+8
**范围**: P0 only — approve 500 → 200 + 前端兼容格式 + 等待派工

---

## 修改文件

| 仓库 | 文件 | 改动 |
|------|------|------|
| ai-company | coding_agent_server.py | 方案格式前端兼容 + approve 用 task_ledger + bridge 返回 executionSession |
| 3d-office | Office3D.tsx | approve 后显示"等待派工"而非假装执行中 |

## 关键变更

### Hub 后端
1. **generate_proposals** — 返回前端 `Proposal` 接口格式：name/core_idea/pros/cons/cost/time/impact/tasks/owner
2. **proposals_approve** — 用 `task_ledger.create_task()` 创建真实任务，读 `proposal.tasks` 兼容前端
3. **bridge_approved_to-board** — 返回 `executionSession.executionId` 满足前端 onApproved 回调

### 3D Office 前端
4. **onApproved** — 不再设置所有 agent 为 "working"（假执行），改为显示 toast："方案已批准：X（N 个任务已创建，等待派工）"

## 验收结果

```
1. generate_proposals: 返回前端兼容格式 ✅
   方案A: 快速收尾 (coding_agent, 低, 2-4h)
   方案B: 质量加固 (qa_agent, 中, 4-6h)
   方案C: 推进主线 (ceo, 低, 持续)

2. approve: HTTP 200, 创建 2 个任务 ✅
   T0513112030-f18f: 分析阻塞根因 → coding_agent
   T0513112030-0300: 限时清除 → coding_agent

3. bridge: HTTP 200, executionSession.exec_1778613630 ✅

4. 前端: approve 后显示 toast "任务已创建，等待派工" ✅
```

## 未完成（P1/P2）

- ❌ 方案内容仍偏通用（非 LLM 实时分析）
- ❌ Agent 发言仍为规则模板，无真实讨论/质询
- ❌ CEO 无主持/汇总/派工动作
- ❌ 无 Telegram 推送
- ❌ 无 3D Office traces 实时状态
- ❌ pm_agent / ux_agent 未加入会议

## 下一步建议

- P1: 方案内容根据真实阻塞/任务数据细化（当前只有骨架）
- P1: Agent 角色差异化发言（pm/ux 加入）
- P2: 两轮质询机制（A 提风险 → B 回应 → CEO 汇总）
- P3: task-ledger → 3D Office traces → Telegram 推送链路
