# OpenClaw 异步派工模型 v1.2

> **核心原则：ceo 不阻塞，ceo 只派工**
> 
> 复用现有 skill，不造新轮子。
> 派单 skill 解决并行，异步队列解决不阻塞。

**版本：** v1.2  
**日期：** 2026-05-13  
**作者：** 千尘 🌌  
**状态：** 设计基线，暂不实现  
**更新：** 基于 v1.1 修正，复用现有派单 skill，新增轻量异步 + 调用日志

---

## 一、设计原则

### 1.1 复用现有 skill

**不要继续造一个完全新的派工系统。**

**应该：**
1. 复用现有 `dispatching-parallel-agents` 处理多独立任务并行
2. 新增轻量异步任务队列，解决 ceo 长任务不阻塞
3. 新增 skill 调用日志，记录每次 skill 是否被识别、是否调用、为什么没调用

### 1.2 两个维度

| 维度 | 问题 | 解决方案 |
|------|------|----------|
| **并行** | 多个独立任务同时执行 | `dispatching-parallel-agents` |
| **异步** | ceo 派工后不等待，继续对话 | 轻量异步任务队列 |

**说明：**
- 并行：多个任务同时跑
- 异步：派工后不阻塞主会话
- 两者可以组合使用

---

## 二、ceo 任务匹配检查

### 2.1 检查流程

```
ceo 接收任务
    ↓
1. 是否有现有 skill 可用？
    ↓ 是 → 使用 skill
    ↓ 否 → 继续检查
2. 是否需要异步？（预计 > 10秒）
    ↓ 是 → 异步派工
    ↓ 否 → 同步执行
3. 是否需要并行？（2+独立任务）
    ↓ 是 → 使用派单 skill
    ↓ 否 → 单一任务
4. 是否需要 Codex？（代码相关）
    ↓ 是 → 派给 coding_agent + Codex
    ↓ 否 → 其他执行层
5. 是否需要人工确认？（高风险）
    ↓ 是 → 等待用户确认
    ↓ 否 → 自动执行
```

### 2.2 检查记录

每次检查都记录到 skill 调用日志：

```json
{
  "timestamp": "2026-05-13T06:20:00+08:00",
  "task": "检查V5记忆系统",
  "checks": {
    "has_skill": true,
    "skill_name": "codex.check_v5",
    "need_async": true,
    "need_parallel": false,
    "need_codex": true,
    "need_human_approval": false
  },
  "decision": "异步派工给 coding_agent + Codex",
  "reason": "预计 > 10秒，单一任务，需要 Codex"
}
```

---

## 三、复用派单 Skill

### 3.1 派单 Skill 现状

**名称：** `dispatching-parallel-agents`

**功能：** 多个独立任务并行派发

**触发条件：** 2+个独立任务，可并行执行

### 3.2 使用场景

**适合使用派单 skill：**
- 用户提出多个不相关的任务
- 系统出现多个独立故障
- 需要同时检查多个方面

**示例：**
- 同时检查 V5、3D办公室、服务状态（三个独立检查）
- 同时修复多个不相关的 bug
- 同时运行多个独立的测试

**不适合使用派单 skill：**
- 任务相互依赖
- 需要完整上下文
- 共享状态

### 3.3 集成方式

**ceo 决策流程：**
1. 识别任务是否可拆分为独立子任务
2. 如果可拆分，使用派单 skill
3. 派单 skill 负责并行调度
4. 异步队列负责不阻塞主会话

---

## 四、轻量异步任务队列

### 4.1 设计目标

**解决：** ceo 长任务不阻塞主会话

**不解决：** 任务并行调度（由派单 skill 解决）

### 4.2 任务状态机

```
                    ┌─────────────┐
                    │   queued    │
                    │   等待派工   │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
              ┌─────│   running   │─────┐
              │     │   执行中    │     │
              │     └─────────────┘     │
              │                         │
              ▼                         ▼
     ┌─────────────────┐      ┌─────────────────┐
     │ waiting_review  │      │     failed      │
     │   等待验收       │      │     失败         │
     └────────┬────────┘      └────────┬────────┘
              │                         │
              ▼                         ▼
     ┌─────────────────┐      ┌─────────────────┐
     │      done       │      │   safe_mode     │
     │     完成         │      │   冻结待处理     │
     └─────────────────┘      └─────────────────┘
```

### 4.3 任务单结构

```json
{
  "task_id": "task-20260513-001",
  "title": "检查V5记忆系统健康状态",
  "description": "只读检查 memory/ 目录结构、MEMORY.md 完整性、索引状态",
  
  "created_by": "ceo",
  "assignee": "coding_agent",
  "tool": "codex",
  "mode": "readonly",
  
  "status": "queued",
  "priority": "normal",
  
  "callback": "telegram:USER_ID_1",
  "session_id": "current",
  
  "created_at": "2026-05-13T05:30:00+08:00",
  "started_at": null,
  "finished_at": null,
  
  "retry_count": 0,
  "max_retries": 3,
  
  "summary": null,
  "output": null,
  "error": null,
  "artifact_path": null,
  
  "safety": {
    "sandbox": "read-only",
    "approval": "never",
    "whitelist": ["codex.check_v5"]
  },
  
  "skill_check": {
    "has_skill": true,
    "skill_name": "codex.check_v5",
    "need_async": true,
    "need_parallel": false,
    "need_codex": true,
    "need_human_approval": false
  }
}
```

### 4.4 派工流程

```
1. ceo 识别任务
   └─ 预计 > 10秒？ → 必须异步
   
2. 创建任务单
   └─ 写入任务队列
   └─ 记录 skill 匹配检查结果
   
3. 选择执行层
   ├─ 代码相关 → coding_agent
   ├─ 工具/自动化 → tools_agent
   └─ 验证/测试 → qa_agent
   
4. 立即回复用户（不等待）
   
5. 主会话释放，继续对话
```

---

## 五、Skill 调用日志

### 5.1 日志格式

**文件：** `skill-call-logs/YYYY-MM-DD.jsonl`

**记录内容：**
```json
{
  "timestamp": "2026-05-13T06:20:00+08:00",
  "session_id": "session-123",
  "task": "检查V5记忆系统",
  "user_message": "检查一下 memory 状态",
  
  "skill_check": {
    "has_skill": true,
    "skill_name": "codex.check_v5",
    "skill_path": "skills/codex-check-v5",
    "trigger_conditions_met": true,
    "trigger_reason": "用户请求检查V5记忆系统"
  },
  
  "async_check": {
    "need_async": true,
    "reason": "预计 > 10秒"
  },
  
  "parallel_check": {
    "need_parallel": false,
    "reason": "单一检查任务"
  },
  
  "codex_check": {
    "need_codex": true,
    "reason": "代码/文件检查"
  },
  
  "approval_check": {
    "need_human_approval": false,
    "reason": "只读检查，低风险"
  },
  
  "decision": "异步派工给 coding_agent + Codex",
  "action": "create_task",
  "task_id": "task-20260513-001"
}
```

### 5.2 日志用途

1. **追溯 skill 调用** - 是否被识别、是否被调用
2. **分析未调用原因** - 为什么某些 skill 未被使用
3. **优化触发条件** - 调整 skill 的触发逻辑
4. **验证调度能力** - 确认 skill 调度是否正常

### 5.3 定期审计

**每周审计：**
1. 检查 skill 调用日志
2. 识别未使用的 skills
3. 分析未使用原因
4. 优化触发条件

---

## 六、正确分工

### 6.1 ceo / 千尘中枢

| 职责 | 说明 |
|------|------|
| 任务识别 | 理解用户意图，识别任务类型 |
| skill 匹配 | 检查是否有现有 skill 可用 |
| 异步判断 | 判断是否需要异步执行 |
| 并行判断 | 判断是否需要并行执行 |
| 派工 | 创建任务单，选择执行层 |
| 验收 | 检查执行结果，决定是否通过 |
| 继续沟通 | 派工后立即回到对话状态 |

### 6.2 dispatching-parallel-agents

| 职责 | 说明 |
|------|------|
| 并行调度 | 多个独立任务并行派发 |
| 任务拆分 | 将大任务拆分为独立子任务 |
| 结果汇总 | 汇总多个执行层的结果 |

### 6.3 轻量异步队列

| 职责 | 说明 |
|------|------|
| 异步执行 | ceo 派工后不等待 |
| 状态管理 | 管理任务状态 |
| 回调通知 | 任务完成后通知 ceo |

### 6.4 Skill 调用日志

| 职责 | 说明 |
|------|------|
| 记录调用 | 记录每次 skill 调用 |
| 分析原因 | 分析为什么某些 skill 未被调用 |
| 定期审计 | 定期审计 skill 使用情况 |

---

## 七、集成方式

### 7.1 在六层架构中的位置

```
用户入口层
    ↓
千尘中枢层 ←───────┐
    │              │
    │ 任务识别      │ 回报结果
    │ skill 匹配    │
    │ 异步判断      │
    │ 并行判断      │
    ↓              │
任务路由与安全控制层 │
    │              │
    │ 派工         │
    ↓              │
执行层 ────────────┘
    │
    ↓
状态与数据层（任务队列、日志、skill调用日志）
    ↓
反馈展示层（Telegram/3D办公室）
```

### 7.2 与现有 skill 集成

**派单 skill：**
- 处理多独立任务并行
- 不处理异步不阻塞

**异步队列：**
- 处理 ceo 长任务不阻塞
- 不处理任务并行

**Skill 调用日志：**
- 记录每次 skill 调用
- 定期审计 skill 使用

---

## 八、第一阶段适配

### 8.1 最小实现

**改动点：**
1. 添加 skill 调用日志
2. 添加 skill 匹配检查流程
3. 轻量异步队列（复用现有任务队列）

**不动点：**
1. 派单 skill 功能不变
2. 现有 skill 不变
3. 白名单任务不变

### 8.2 实现顺序

1. **第一阶段：** 添加 skill 调用日志
2. **第二阶段：** 添加 skill 匹配检查
3. **第三阶段：** 轻量异步队列优化

---

## 九、战略意义

**不造新轮子，复用现有 skill。**

**核心改进：**
1. 复用派单 skill 处理并行
2. 新增轻量异步处理不阻塞
3. 新增调用日志验证调度能力

**这才是真正的"第二大脑"体验。**

---

## 更新日志

### v1.2 (2026-05-13)
- 基于 v1.1 修正
- 复用现有派单 skill
- 新增轻量异步任务队列
- 新增 skill 调用日志
- 新增 ceo 任务匹配检查

### v1.1 (2026-05-13)
- 基于 v1.0 优化
- 新增异步触发机制

### v1.0 (2026-05-13)
- 初始设计
- 定义任务状态机、任务单结构、派工协议、回报协议、安全规则

---

*设计文档（修正版） — 2026-05-13*
