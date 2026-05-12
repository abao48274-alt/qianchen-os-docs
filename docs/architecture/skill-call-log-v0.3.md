# Skill 调用日志 v0.3 设计

> **核心目标：让 ceo 每次任务决策可追踪**
> 
> 只记录，不改变现有行为。
> 分本地完整日志 + 公共脱敏日志。
> 敏感信息不进公共仓库。
> 只记录任务决策节点。

**版本：** v0.3  
**日期：** 2026-05-13  
**作者：** 千尘 🌌  
**状态：** 设计基线，暂不实现  
**更新：** 基于 v0.2 修正，去掉 session_hash、二次脱敏、枚举化、明确记录范围

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

### 1.2 双日志架构

**本地完整日志：**
- 位置：`ai-company/skill-call-logs/`
- 内容：完整记录，包含所有字段
- 用途：ceo 内部追溯、调试、审计
- 安全：仅本机存储，不推送

**公共脱敏日志：**
- 位置：`qianchen-os-docs/task-logs/public/skill-calls/`
- 内容：脱敏后记录，仅保留必要信息
- 用途：公开审计、协作、分析
- 安全：脱敏后推送，无敏感信息

### 1.3 可追溯性

**目标：** 让 ceo 的每次任务决策都有记录可查。

**用途：**
1. 追溯 skill 调用情况
2. 分析为什么某些 skill 未被使用
3. 验证 skill 调度能力
4. 优化触发条件

---

## 二、记录范围

### 2.1 记录的任务决策节点

**只记录以下情况：**

1. **用户提出任务请求**
   - "检查一下..."
   - "帮我做..."
   - "派谁去做..."
   - "修复这个问题..."

2. **ceo 识别到系统任务**
   - 定时任务触发
   - 系统状态异常
   - 监控告警

3. **ceo 做出任务决策**
   - 决定是否使用 skill
   - 决定是否异步
   - 决定是否并行
   - 决定是否使用 Codex
   - 决定是否需要人工确认

### 2.2 不记录的情况

**以下情况不记录：**

1. **普通聊天**
   - "你好"
   - "今天怎么样"
   - "谢谢"

2. **普通问答**
   - "什么是..."
   - "怎么用..."
   - "为什么..."

3. **闲聊**
   - 日常对话
   - 非任务讨论
   - 信息分享

4. **信息查询**
   - "查一下..."
   - "告诉我..."
   - "解释一下..."

**判断标准：** 是否涉及任务执行决策

---

## 三、脱敏规则

### 3.1 禁止进入公共仓库的内容

| 禁止项 | 原因 | 处理方式 |
|--------|------|----------|
| `user_message` 原文 | 可能包含隐私 | 删除 |
| 真实 `session_id` | 可追溯用户 | 删除 |
| `session_hash` | 第一阶段不记录 | 删除 |
| Telegram ID | 用户隐私 | 删除 |
| 本机绝对路径 | 系统信息泄露 | 删除 |
| 私有配置 | 安全风险 | 删除 |
| API Token / Key | 安全风险 | 删除 |
| 密码 / Secret | 安全风险 | 删除 |
| 人名 | 隐私 | 删除 |
| 账号 | 隐私 | 删除 |
| 私有仓库名 | 安全风险 | 删除 |

### 3.2 公共日志保留内容

**公共日志保留：**
- `timestamp`（时间）
- `channel_type`（渠道类型）
- `task_category`（任务类别）
- `skill_name`（skill 名称）
- `skill_matched`（是否匹配）
- `skill_called`（是否调用）
- `not_called_reason`（未调用原因 - 枚举）
- `need_async`（是否需要异步）
- `need_parallel`（是否需要并行）
- `need_codex`（是否需要 Codex）
- `need_human_approval`（是否需要人工确认）
- `risk_level`（风险等级）
- `decision`（最终决策 - 枚举）

### 3.3 二次脱敏

**task_summary → task_category：**
- 原始：`检查一下 memory 状态`
- 一次脱敏：`检查V5记忆系统`
- 二次脱敏：`系统检查`

**示例映射：**
| 原始任务 | 一次脱敏 | 二次脱敏（类别） |
|----------|----------|------------------|
| 检查一下 memory 状态 | 检查V5记忆系统 | 系统检查 |
| 帮我修复这个 bug | 修复登录模块 bug | bug 修复 |
| 派 coding_agent 去改代码 | 修改用户模块代码 | 代码修改 |
| 同时检查三个系统 | 三个系统并行检查 | 批量检查 |
| 做一个架构设计 | 设计异步派工模型 | 架构设计 |

---

## 四、枚举定义

### 4.1 not_called_reason 枚举

**枚举值：**

| 枚举值 | 含义 |
|--------|------|
| `NO_MATCH` | 没有匹配的 skill |
| `ASYNC_REQUIRED` | 任务需要异步，skill 不支持 |
| `SEQUENTIAL_TASK` | 串行任务，不适合并行 |
| `CONTEXT_REQUIRED` | 需要完整上下文 |
| `SHARED_STATE` | 有共享状态，不适合并行 |
| `LOW_PRIORITY` | 低优先级，暂不处理 |
| `MANUAL_APPROVAL_REQUIRED` | 需要人工确认 |
| `SKILL_DEPRECATED` | skill 已废弃 |
| `OTHER` | 其他原因 |

### 4.2 decision 枚举

**枚举值：**

| 枚举值 | 含义 |
|--------|------|
| `SYNC_EXECUTE` | 同步执行 |
| `ASYNC_DISPATCH` | 异步派工 |
| `PARALLEL_DISPATCH` | 并行派工 |
| `HUMAN_APPROVAL` | 等待人工确认 |
| `SKIP` | 跳过，不执行 |
| `DEFER` | 延后处理 |
| `ESCALATE` | 升级处理 |

### 4.3 channel_type 枚举

**枚举值：**

| 枚举值 | 含义 |
|--------|------|
| `TELEGRAM` | Telegram 渠道 |
| `DIRECT` | 直连（OpenClaw 主会话） |
| `3D_OFFICE` | 3D 办公室 |
| `APP` | 未来 App |
| `CRON` | 定时任务 |
| `SYSTEM` | 系统触发 |

### 4.4 risk_level 枚举

**枚举值：**

| 枚举值 | 含义 |
|--------|------|
| `LOW` | 低风险（只读、检查） |
| `MEDIUM` | 中风险（写入、修改） |
| `HIGH` | 高风险（删除、发布） |
| `CRITICAL` | 极高风险（系统配置、密钥） |

---

## 五、记录内容

### 5.1 本地完整日志

**文件：** `ai-company/skill-call-logs/YYYY-MM-DD.jsonl`

**记录字段：**

| 字段 | 说明 | 示例 |
|------|------|------|
| `timestamp` | 决策时间 | `2026-05-13T06:25:00+08:00` |
| `session_id` | 会话 ID | `session-abc123` |
| `channel_type` | 渠道类型 | `TELEGRAM` |
| `user_message` | 用户原始消息 | `检查一下 memory 状态` |
| `task_description` | ceo 理解的任务 | `检查V5记忆系统健康状态` |
| `task_category` | 任务类别 | `系统检查` |
| `skill_check` | skill 匹配检查（完整） | 见下文 |
| `async_check` | 异步判断 | 见下文 |
| `parallel_check` | 并行判断 | 见下文 |
| `codex_check` | Codex 判断 | 见下文 |
| `approval_check` | 人工确认判断 | 见下文 |
| `decision` | 最终决策（枚举） | `ASYNC_DISPATCH` |
| `decision_detail` | 决策详情 | `异步派工给 coding_agent + Codex` |
| `action` | 实际动作 | `create_task` |
| `task_id` | 任务 ID | `task-20260513-001` |

**skill_check 完整结构：**
```json
{
  "has_skill": true,
  "skill_name": "codex.check_v5",
  "skill_path": "skills/codex-check-v5",
  "skill_matched": true,
  "skill_called": false,
  "not_called_reason": "ASYNC_REQUIRED"
}
```

### 5.2 公共脱敏日志

**文件：** `qianchen-os-docs/task-logs/public/skill-calls/YYYY-MM-DD.jsonl`

**记录字段：**

| 字段 | 说明 | 示例 |
|------|------|------|
| `timestamp` | 决策时间 | `2026-05-13T06:25:00+08:00` |
| `channel_type` | 渠道类型 | `TELEGRAM` |
| `task_category` | 任务类别 | `系统检查` |
| `skill_name` | skill 名称 | `codex.check_v5` |
| `skill_matched` | 是否匹配 | `true` |
| `skill_called` | 是否调用 | `false` |
| `not_called_reason` | 未调用原因（枚举） | `ASYNC_REQUIRED` |
| `need_async` | 是否需要异步 | `true` |
| `need_parallel` | 是否需要并行 | `false` |
| `need_codex` | 是否需要 Codex | `true` |
| `need_human_approval` | 是否需要人工确认 | `false` |
| `risk_level` | 风险等级 | `LOW` |
| `decision` | 最终决策（枚举） | `ASYNC_DISPATCH` |

---

## 六、日志格式

### 6.1 本地完整日志格式

**文件位置：**
```
ai-company/
└── skill-call-logs/
    └── YYYY-MM-DD.jsonl
```

**示例日志条目：**
```json
{
  "timestamp": "2026-05-13T06:25:00+08:00",
  "session_id": "session-abc123",
  "channel_type": "TELEGRAM",
  "user_message": "检查一下 memory 状态",
  "task_description": "检查V5记忆系统健康状态",
  "task_category": "系统检查",
  
  "skill_check": {
    "has_skill": true,
    "skill_name": "codex.check_v5",
    "skill_path": "skills/codex-check-v5",
    "skill_matched": true,
    "skill_called": false,
    "not_called_reason": "ASYNC_REQUIRED"
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
    "risk_level": "LOW"
  },
  
  "decision": "ASYNC_DISPATCH",
  "decision_detail": "异步派工给 coding_agent + Codex",
  "action": "create_task",
  "task_id": "task-20260513-001"
}
```

### 6.2 公共脱敏日志格式

**文件位置：**
```
qianchen-os-docs/
└── task-logs/
    └── public/
        └── skill-calls/
            └── YYYY-MM-DD.jsonl
```

**示例日志条目：**
```json
{
  "timestamp": "2026-05-13T06:25:00+08:00",
  "channel_type": "TELEGRAM",
  "task_category": "系统检查",
  "skill_name": "codex.check_v5",
  "skill_matched": true,
  "skill_called": false,
  "not_called_reason": "ASYNC_REQUIRED",
  "need_async": true,
  "need_parallel": false,
  "need_codex": true,
  "need_human_approval": false,
  "risk_level": "LOW",
  "decision": "ASYNC_DISPATCH"
}
```

### 6.3 JSONL 格式

每行一个 JSON 对象，方便追加和查询。

**本地日志文件内容：**
```
{"timestamp":"2026-05-13T06:25:00+08:00","session_id":"session-abc123","channel_type":"TELEGRAM","user_message":"检查一下 memory 状态",...}
{"timestamp":"2026-05-13T06:30:00+08:00","session_id":"session-def456","channel_type":"DIRECT","user_message":"同时检查三个系统",...}
```

**公共日志文件内容：**
```
{"timestamp":"2026-05-13T06:25:00+08:00","channel_type":"TELEGRAM","task_category":"系统检查","skill_name":"codex.check_v5",...}
{"timestamp":"2026-05-13T06:30:00+08:00","channel_type":"DIRECT","task_category":"批量检查","skill_name":"dispatching-parallel-agents",...}
```

---

## 七、记录时机

### 7.1 记录触发点

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

### 7.2 不记录的情况

**以下情况不记录：**

1. **纯聊天** - 没有任务需求
2. **信息查询** - 只是问问题
3. **闲聊** - 不涉及任务执行

---

## 八、脱敏流程

### 8.1 本地记录 → 公共记录

**流程：**
1. ceo 决策时记录本地完整日志
2. 定期（如每天）将本地日志脱敏为公共日志
3. 脱敏后推送到公共仓库
4. 本地完整日志保留，不推送

### 8.2 脱敏规则

**字段脱敏：**
- `user_message` → 删除
- `session_id` → 删除
- `task_description` → 二次脱敏为 `task_category`
- `decision_detail` → 删除，只保留 `decision` 枚举
- 路径 → 删除
- 私有配置 → 删除

**枚举转换：**
- `not_called_reason` → 使用枚举值
- `decision` → 使用枚举值
- `risk_level` → 使用枚举值

### 8.3 任务类别映射

**常见任务类别：**

| 任务类别 | 示例任务 |
|----------|----------|
| `系统检查` | 检查 V5、检查服务状态、检查 3D 办公室 |
| `bug 修复` | 修复登录 bug、修复显示问题 |
| `代码修改` | 修改用户模块、重构代码 |
| `批量检查` | 同时检查多个系统 |
| `架构设计` | 设计异步派工模型、设计新架构 |
| `文档编写` | 编写 README、编写 API 文档 |
| `测试执行` | 运行单元测试、运行集成测试 |
| `部署发布` | 部署到生产环境、发布新版本 |
| `监控告警` | 处理告警、检查监控 |
| `其他` | 未分类任务 |

---

## 九、使用方式

### 9.1 手动记录

**ceo 在决策时手动记录：**

```python
# 伪代码示例
def log_skill_call(user_message, task_description, skill_info, decision):
    # 本地完整日志
    local_log_entry = {
        "timestamp": get_timestamp(),
        "session_id": get_session_id(),
        "channel_type": get_channel_type(),
        "user_message": user_message,
        "task_description": task_description,
        "task_category": categorize_task(task_description),
        "skill_check": skill_info,
        "decision": decision,
        "decision_detail": get_decision_detail(decision)
    }
    append_to_local_log_file(local_log_entry)
    
    # 公共脱敏日志（定期生成）
    # public_log_entry = desensitize(local_log_entry)
    # append_to_public_log_file(public_log_entry)
```

### 9.2 记录位置

**本地完整日志：**
- `ai-company/skill-call-logs/YYYY-MM-DD.jsonl`
- 包含完整信息，不推送

**公共脱敏日志：**
- `qianchen-os-docs/task-logs/public/skill-calls/YYYY-MM-DD.jsonl`
- 脱敏后推送

### 9.3 查询方式

**按日期查询（本地）：**
```bash
cat ai-company/skill-call-logs/2026-05-13.jsonl | jq .
```

**按 skill 名称查询（公共）：**
```bash
cat qianchen-os-docs/task-logs/public/skill-calls/*.jsonl | jq 'select(.skill_name == "codex.check_v5")'
```

**按决策类型查询（公共）：**
```bash
cat qianchen-os-docs/task-logs/public/skill-calls/*.jsonl | jq 'select(.decision == "ASYNC_DISPATCH")'
```

---

## 十、分析用途

### 10.1 追溯 skill 调用

**问题：** 这个 skill 被调用过吗？

**查询（公共日志）：**
```bash
cat qianchen-os-docs/task-logs/public/skill-calls/*.jsonl | jq 'select(.skill_name == "skill-name")'
```

### 10.2 分析未调用原因

**问题：** 为什么这个 skill 没有被调用？

**查询（公共日志）：**
```bash
cat qianchen-os-docs/task-logs/public/skill-calls/*.jsonl | jq 'select(.skill_called == false)'
```

### 10.3 验证调度能力

**问题：** skill 调度能力正常吗？

**分析：**
1. 检查是否有 skill 被识别但未调用
2. 检查未调用原因是否合理
3. 检查是否有 skill 应该被使用但未被识别

### 10.4 优化触发条件

**问题：** skill 的触发条件需要调整吗？

**分析：**
1. 检查 skill 被识别的频率
2. 检查 skill 被调用的频率
3. 检查未调用原因是否合理
4. 调整触发条件

---

## 十一、与现有系统的集成

### 11.1 不改变现有行为

**ceo 继续按现有方式工作：**
- 继续识别任务
- 继续判断是否使用 skill
- 继续派工
- 继续汇报

**只添加记录：**
- 在决策时记录日志
- 不改变决策过程
- 不改变执行方式

### 11.2 记录位置

**本地完整日志：**
- `ai-company/skill-call-logs/YYYY-MM-DD.jsonl`

**公共脱敏日志：**
- `qianchen-os-docs/task-logs/public/skill-calls/YYYY-MM-DD.jsonl`

### 11.3 定期审计

**每周审计：**
1. 检查 skill 调用日志
2. 识别未使用的 skills
3. 分析未使用原因
4. 优化触发条件

---

## 十二、最小实现

### 12.1 第一阶段：手动记录

**实现方式：**
- ceo 在决策时手动创建日志条目
- 保存到 `ai-company/skill-call-logs/` 目录
- 定期脱敏并备份到公共仓库

**不做的事：**
- ❌ 不自动记录
- ❌ 不解析日志
- ❌ 不生成报告

### 12.2 第二阶段：半自动记录

**实现方式：**
- ceo 在决策时调用记录函数
- 函数自动填充基础信息
- ceo 手动补充决策信息

**不做的事：**
- ❌ 不自动触发记录
- ❌ 不自动分析日志

### 12.3 第三阶段：全自动记录

**实现方式：**
- 系统自动检测任务决策
- 系统自动填充所有信息
- 系统自动保存日志

**不做的事：**
- ❌ 不改变决策过程
- ❌ 不自动执行任务

---

## 十三、示例场景

### 13.1 场景 1：检查 V5 记忆系统

**用户消息：** `检查一下 memory 状态`

**ceo 决策过程：**
1. 识别任务：检查V5记忆系统
2. 检查 skill：有 `codex.check_v5`
3. 判断异步：需要，预计 > 10秒
4. 判断并行：不需要，单一任务
5. 判断 Codex：需要，检查文件系统
6. 判断人工确认：不需要，只读低风险
7. 最终决策：异步派工给 coding_agent + Codex

**本地完整日志：**
```json
{
  "timestamp": "2026-05-13T06:25:00+08:00",
  "session_id": "session-abc123",
  "channel_type": "TELEGRAM",
  "user_message": "检查一下 memory 状态",
  "task_description": "检查V5记忆系统健康状态",
  "task_category": "系统检查",
  "skill_check": {
    "has_skill": true,
    "skill_name": "codex.check_v5",
    "skill_path": "skills/codex-check-v5",
    "skill_matched": true,
    "skill_called": false,
    "not_called_reason": "ASYNC_REQUIRED"
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
    "risk_level": "LOW"
  },
  "decision": "ASYNC_DISPATCH",
  "decision_detail": "异步派工给 coding_agent + Codex",
  "action": "create_task",
  "task_id": "task-20260513-001"
}
```

**公共脱敏日志：**
```json
{
  "timestamp": "2026-05-13T06:25:00+08:00",
  "channel_type": "TELEGRAM",
  "task_category": "系统检查",
  "skill_name": "codex.check_v5",
  "skill_matched": true,
  "skill_called": false,
  "not_called_reason": "ASYNC_REQUIRED",
  "need_async": true,
  "need_parallel": false,
  "need_codex": true,
  "need_human_approval": false,
  "risk_level": "LOW",
  "decision": "ASYNC_DISPATCH"
}
```

### 13.2 场景 2：同时检查三个系统

**用户消息：** `同时检查 V5、3D办公室、服务状态`

**ceo 决策过程：**
1. 识别任务：三个独立检查
2. 检查 skill：有三个 skill
3. 判断异步：需要，每个预计 > 10秒
4. 判断并行：需要，三个独立任务
5. 判断 Codex：需要，检查文件系统
6. 判断人工确认：不需要，只读低风险
7. 最终决策：使用派单 skill 并行派工

**本地完整日志：**
```json
{
  "timestamp": "2026-05-13T06:30:00+08:00",
  "session_id": "session-def456",
  "channel_type": "DIRECT",
  "user_message": "同时检查 V5、3D办公室、服务状态",
  "task_description": "三个独立系统检查",
  "task_category": "批量检查",
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
    "risk_level": "LOW"
  },
  "decision": "PARALLEL_DISPATCH",
  "decision_detail": "使用派单 skill 并行派工给三个 coding_agent",
  "action": "dispatch_parallel",
  "task_id": "task-20260513-002"
}
```

**公共脱敏日志：**
```json
{
  "timestamp": "2026-05-13T06:30:00+08:00",
  "channel_type": "DIRECT",
  "task_category": "批量检查",
  "skill_name": "dispatching-parallel-agents",
  "skill_matched": true,
  "skill_called": true,
  "not_called_reason": null,
  "need_async": true,
  "need_parallel": true,
  "need_codex": true,
  "need_human_approval": false,
  "risk_level": "LOW",
  "decision": "PARALLEL_DISPATCH"
}
```

### 13.3 场景 3：普通聊天（不记录）

**用户消息：** `今天天气怎么样？`

**ceo 响应：** 直接回答，不记录日志

**原因：** 这是普通问答，不是任务决策

---

## 十四、总结

### 14.1 设计要点

1. **只记录，不改变** - 不改变 ceo 的现有行为
2. **双日志架构** - 本地完整日志 + 公共脱敏日志
3. **脱敏规则** - 敏感信息不进公共仓库
4. **枚举化** - reason 和 decision 使用枚举值
5. **明确范围** - 只记录任务决策节点

### 14.2 核心价值

1. **验证 skill 调度能力** - 确认 skill 是否被正确识别和调用
2. **分析未调用原因** - 了解为什么某些 skill 未被使用
3. **优化触发条件** - 调整 skill 的触发逻辑
4. **建立决策基线** - 为后续自动化建立基础

### 14.3 下一步

1. **设计通过** - 等待雨陌确认
2. **最小实现** - ceo 手动记录日志
3. **定期审计** - 每周检查日志
4. **逐步优化** - 根据日志优化触发条件

---

*设计文档（枚举化版） — 2026-05-13*
