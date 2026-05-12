# Skill 调用日志 v0.1 设计

> **核心目标：让 ceo 每次任务决策可追踪**
> 
> 只记录，不改变现有行为。
> 不做任务队列，不做 Codex Runner，不做回调，不做自动执行。

**版本：** v0.1  
**日期：** 2026-05-13  
**作者：** 千尘 🌌  
**状态：** 设计，不执行

---

## 一、设计原则

### 1.1 只记录，不改变

**核心原则：** 只记录 ceo 的决策过程，不改变 ceo 的现有行为。

**不做的事：**
- ❌ 不做任务队列
- ❌ 不做 Codex Runner
- ❌ 不做 Telegram 回调
- ❌ 不做任何自动执行
- ❌ 不改变 ceo 的派工方式
- ❌ 不改变 ceo 的汇报方式

**做的事：**
- ✅ 记录 ceo 是否识别了可用 skill
- ✅ 记录 ceo 是否调用了 skill
- ✅ 记录 ceo 为什么没有调用 skill
- ✅ 记录 ceo 的其他决策判断

### 1.2 可追溯性

**目标：** 让 ceo 的每次任务决策都有记录可查。

**用途：**
1. 追溯 skill 调用情况
2. 分析为什么某些 skill 未被使用
3. 验证 skill 调度能力
4. 优化触发条件

---

## 二、记录内容

### 2.1 基础信息

| 字段 | 说明 | 示例 |
|------|------|------|
| `timestamp` | 决策时间 | `2026-05-13T06:25:00+08:00` |
| `session_id` | 会话 ID | `session-abc123` |
| `user_message` | 用户原始消息 | `检查一下 memory 状态` |
| `task_description` | ceo 理解的任务 | `检查V5记忆系统健康状态` |

### 2.2 Skill 匹配检查

| 字段 | 说明 | 示例 |
|------|------|------|
| `has_skill` | 是否有可用 skill | `true` |
| `skill_name` | skill 名称 | `codex.check_v5` |
| `skill_path` | skill 路径 | `skills/codex-check-v5` |
| `skill_matched` | 是否匹配任务 | `true` |
| `skill_called` | 是否调用了 skill | `false` |
| `not_called_reason` | 未调用原因 | `任务需要异步，但 skill 不支持异步` |

### 2.3 异步判断

| 字段 | 说明 | 示例 |
|------|------|------|
| `need_async` | 是否需要异步 | `true` |
| `async_reason` | 为什么需要异步 | `预计 > 10秒` |
| `estimated_duration` | 预计耗时（秒） | `30` |

### 2.4 并行判断

| 字段 | 说明 | 示例 |
|------|------|------|
| `need_parallel` | 是否需要并行 | `false` |
| `parallel_reason` | 为什么需要/不需要并行 | `单一检查任务` |
| `parallel_tasks` | 并行任务列表 | `[]` |

### 2.5 Codex 判断

| 字段 | 说明 | 示例 |
|------|------|------|
| `need_codex` | 是否需要 Codex | `true` |
| `codex_reason` | 为什么需要/不需要 Codex | `需要检查文件系统` |
| `codex_mode` | Codex 模式 | `readonly` |

### 2.6 人工确认判断

| 字段 | 说明 | 示例 |
|------|------|------|
| `need_human_approval` | 是否需要人工确认 | `false` |
| `approval_reason` | 为什么需要/不需要人工确认 | `只读检查，低风险` |
| `risk_level` | 风险等级 | `low` |

### 2.7 最终决策

| 字段 | 说明 | 示例 |
|------|------|------|
| `decision` | ceo 的最终决策 | `异步派工给 coding_agent + Codex` |
| `action` | 实际执行的动作 | `create_task` |
| `task_id` | 任务 ID（如有） | `task-20260513-001` |

---

## 三、日志格式

### 3.1 文件位置

```
ai-company/
└── skill-call-logs/
    └── YYYY-MM-DD.jsonl
```

**示例：**
```
ai-company/skill-call-logs/2026-05-13.jsonl
```

### 3.2 日志条目格式

```json
{
  "timestamp": "2026-05-13T06:25:00+08:00",
  "session_id": "session-abc123",
  "user_message": "检查一下 memory 状态",
  "task_description": "检查V5记忆系统健康状态",
  
  "skill_check": {
    "has_skill": true,
    "skill_name": "codex.check_v5",
    "skill_path": "skills/codex-check-v5",
    "skill_matched": true,
    "skill_called": false,
    "not_called_reason": "任务需要异步，但 skill 不支持异步"
  },
  
  "async_check": {
    "need_async": true,
    "async_reason": "预计 > 10秒",
    "estimated_duration": 30
  },
  
  "parallel_check": {
    "need_parallel": false,
    "parallel_reason": "单一检查任务",
    "parallel_tasks": []
  },
  
  "codex_check": {
    "need_codex": true,
    "codex_reason": "需要检查文件系统",
    "codex_mode": "readonly"
  },
  
  "approval_check": {
    "need_human_approval": false,
    "approval_reason": "只读检查，低风险",
    "risk_level": "low"
  },
  
  "decision": "异步派工给 coding_agent + Codex",
  "action": "create_task",
  "task_id": "task-20260513-001"
}
```

### 3.3 JSONL 格式

每行一个 JSON 对象，方便追加和查询。

**示例文件内容：**
```
{"timestamp":"2026-05-13T06:25:00+08:00","session_id":"session-abc123",...}
{"timestamp":"2026-05-13T06:30:00+08:00","session_id":"session-def456",...}
{"timestamp":"2026-05-13T06:35:00+08:00","session_id":"session-ghi789",...}
```

---

## 四、记录时机

### 4.1 记录触发点

**每次 ceo 做出任务决策时记录：**

1. **用户提出任务请求时**
   - 用户说"检查一下..."
   - 用户说"帮我做..."
   - 用户说"派谁去做..."

2. **ceo 识别到任务时**
   - 从对话中识别出任务
   - 从上下文推断出任务
   - 从系统状态发现任务

3. **ceo 做出决策时**
   - 决定是否使用 skill
   - 决定是否异步
   - 决定是否并行
   - 决定是否使用 Codex
   - 决定是否需要人工确认

### 4.2 不记录的情况

**以下情况不记录：**

1. **纯聊天** - 没有任务需求
2. **信息查询** - 只是问问题
3. **闲聊** - 不涉及任务执行

---

## 五、使用方式

### 5.1 手动记录

**ceo 在决策时手动记录：**

```python
# 伪代码示例
def log_skill_call(user_message, task_description, skill_info, decision):
    log_entry = {
        "timestamp": get_timestamp(),
        "session_id": get_session_id(),
        "user_message": user_message,
        "task_description": task_description,
        "skill_check": skill_info,
        "decision": decision
    }
    append_to_log_file(log_entry)
```

### 5.2 记录位置

**日志文件：**
- `ai-company/skill-call-logs/YYYY-MM-DD.jsonl`

**备份位置：**
- `qianchen-os-docs/task-logs/public/skill-calls/`

### 5.3 查询方式

**按日期查询：**
```bash
cat ai-company/skill-call-logs/2026-05-13.jsonl | jq .
```

**按 skill 名称查询：**
```bash
cat ai-company/skill-call-logs/*.jsonl | jq 'select(.skill_check.skill_name == "codex.check_v5")'
```

**按决策类型查询：**
```bash
cat ai-company/skill-call-logs/*.jsonl | jq 'select(.decision | contains("异步"))'
```

---

## 六、分析用途

### 6.1 追溯 skill 调用

**问题：** 这个 skill 被调用过吗？

**查询：**
```bash
cat ai-company/skill-call-logs/*.jsonl | jq 'select(.skill_check.skill_name == "skill-name")'
```

### 6.2 分析未调用原因

**问题：** 为什么这个 skill 没有被调用？

**查询：**
```bash
cat ai-company/skill-call-logs/*.jsonl | jq 'select(.skill_check.skill_called == false)'
```

### 6.3 验证调度能力

**问题：** skill 调度能力正常吗？

**分析：**
1. 检查是否有 skill 被识别但未调用
2. 检查未调用原因是否合理
3. 检查是否有 skill 应该被使用但未被识别

### 6.4 优化触发条件

**问题：** skill 的触发条件需要调整吗？

**分析：**
1. 检查 skill 被识别的频率
2. 检查 skill 被调用的频率
3. 检查未调用原因是否合理
4. 调整触发条件

---

## 七、与现有系统的集成

### 7.1 不改变现有行为

**ceo 继续按现有方式工作：**
- 继续识别任务
- 继续判断是否使用 skill
- 继续派工
- 继续汇报

**只添加记录：**
- 在决策时记录日志
- 不改变决策过程
- 不改变执行方式

### 7.2 记录位置

**本地日志：**
- `ai-company/skill-call-logs/YYYY-MM-DD.jsonl`

**公共仓库：**
- `qianchen-os-docs/task-logs/public/skill-calls/YYYY-MM-DD.jsonl`（脱敏版本）

### 7.3 定期审计

**每周审计：**
1. 检查 skill 调用日志
2. 识别未使用的 skills
3. 分析未使用原因
4. 优化触发条件

---

## 八、最小实现

### 8.1 第一阶段：手动记录

**实现方式：**
- ceo 在决策时手动创建日志条目
- 保存到 `ai-company/skill-call-logs/` 目录
- 定期备份到公共仓库

**不做的事：**
- ❌ 不自动记录
- ❌ 不解析日志
- ❌ 不生成报告

### 8.2 第二阶段：半自动记录

**实现方式：**
- ceo 在决策时调用记录函数
- 函数自动填充基础信息
- ceo 手动补充决策信息

**不做的事：**
- ❌ 不自动触发记录
- ❌ 不自动分析日志

### 8.3 第三阶段：全自动记录

**实现方式：**
- 系统自动检测任务决策
- 系统自动填充所有信息
- 系统自动保存日志

**不做的事：**
- ❌ 不改变决策过程
- ❌ 不自动执行任务

---

## 九、示例场景

### 9.1 场景 1：检查 V5 记忆系统

**用户消息：** `检查一下 memory 状态`

**ceo 决策过程：**
1. 识别任务：检查V5记忆系统
2. 检查 skill：有 `codex.check_v5`
3. 判断异步：需要，预计 > 10秒
4. 判断并行：不需要，单一任务
5. 判断 Codex：需要，检查文件系统
6. 判断人工确认：不需要，只读低风险
7. 最终决策：异步派工给 coding_agent + Codex

**日志条目：**
```json
{
  "timestamp": "2026-05-13T06:25:00+08:00",
  "session_id": "session-abc123",
  "user_message": "检查一下 memory 状态",
  "task_description": "检查V5记忆系统健康状态",
  
  "skill_check": {
    "has_skill": true,
    "skill_name": "codex.check_v5",
    "skill_path": "skills/codex-check-v5",
    "skill_matched": true,
    "skill_called": false,
    "not_called_reason": "任务需要异步，但 skill 不支持异步"
  },
  
  "async_check": {
    "need_async": true,
    "async_reason": "预计 > 10秒",
    "estimated_duration": 30
  },
  
  "parallel_check": {
    "need_parallel": false,
    "parallel_reason": "单一检查任务",
    "parallel_tasks": []
  },
  
  "codex_check": {
    "need_codex": true,
    "codex_reason": "需要检查文件系统",
    "codex_mode": "readonly"
  },
  
  "approval_check": {
    "need_human_approval": false,
    "approval_reason": "只读检查，低风险",
    "risk_level": "low"
  },
  
  "decision": "异步派工给 coding_agent + Codex",
  "action": "create_task",
  "task_id": "task-20260513-001"
}
```

### 9.2 场景 2：同时检查三个系统

**用户消息：** `同时检查 V5、3D办公室、服务状态`

**ceo 决策过程：**
1. 识别任务：三个独立检查
2. 检查 skill：有三个 skill
3. 判断异步：需要，每个预计 > 10秒
4. 判断并行：需要，三个独立任务
5. 判断 Codex：需要，检查文件系统
6. 判断人工确认：不需要，只读低风险
7. 最终决策：使用派单 skill 并行派工

**日志条目：**
```json
{
  "timestamp": "2026-05-13T06:30:00+08:00",
  "session_id": "session-def456",
  "user_message": "同时检查 V5、3D办公室、服务状态",
  "task_description": "三个独立系统检查",
  
  "skill_check": {
    "has_skill": true,
    "skill_name": "dispatching-parallel-agents",
    "skill_path": "skills/dispatching-parallel-agents",
    "skill_matched": true,
    "skill_called": true,
    "not_called_reason": null
  },
  
  "async_check": {
    "need_async": true,
    "async_reason": "每个检查预计 > 10秒",
    "estimated_duration": 90
  },
  
  "parallel_check": {
    "need_parallel": true,
    "parallel_reason": "三个独立检查任务",
    "parallel_tasks": [
      "codex.check_v5",
      "codex.check_office",
      "codex.check_services"
    ]
  },
  
  "codex_check": {
    "need_codex": true,
    "codex_reason": "需要检查文件系统",
    "codex_mode": "readonly"
  },
  
  "approval_check": {
    "need_human_approval": false,
    "approval_reason": "只读检查，低风险",
    "risk_level": "low"
  },
  
  "decision": "使用派单 skill 并行派工给三个 coding_agent",
  "action": "dispatch_parallel",
  "task_id": "task-20260513-002"
}
```

---

## 十、总结

### 10.1 设计要点

1. **只记录，不改变** - 不改变 ceo 的现有行为
2. **可追溯** - 每次决策都有记录可查
3. **轻量** - 只记录必要信息
4. **手动** - ceo 手动记录，不自动执行

### 10.2 核心价值

1. **验证 skill 调度能力** - 确认 skill 是否被正确识别和调用
2. **分析未调用原因** - 了解为什么某些 skill 未被使用
3. **优化触发条件** - 调整 skill 的触发逻辑
4. **建立决策基线** - 为后续自动化建立基础

### 10.3 下一步

1. **设计通过** - 等待雨陌确认
2. **最小实现** - ceo 手动记录日志
3. **定期审计** - 每周检查日志
4. **逐步优化** - 根据日志优化触发条件

---

*设计文档 — 2026-05-13*
