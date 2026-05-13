# Agent 派工握手机制 v1

**版本**：1.0  
**建立时间**：2026-05-14 05:55 GMT+8  
**建立者**：千尘（CEO）  
**状态**：正式生效

---

## 1. 问题背景

### 1.1 当前事故
- 04:55 已派工给 tools_agent/qa_agent/memory_agent
- 05:40 检查：无完成消息、无产出文件
- 子Agent 最后活跃：04:54（派工前）
- **结论**：派工没有形成接单确认，CEO 失去监督闭环

### 1.2 根本原因
- 派工只发消息，不等确认
- 无 ACK 机制，无法确认 Agent 收到任务
- 无状态跟踪，无法知道任务进度
- 无超时处理，Agent 不响应时无升级机制

---

## 2. 派工握手机制

### 2.1 必须包含的要素
1. **接单确认**：Agent 必须回复 ACK
2. **明确 owner**：每个任务指定唯一负责 Agent
3. **明确 deadline**：每个任务指定截止时间
4. **明确产出文件**：每个任务指定产出路径
5. **明确验收标准**：每个任务指定验收条件
6. **超时升级**：未确认/未完成时的处理机制

### 2.2 状态定义
| 状态 | 含义 | 触发条件 |
|------|------|----------|
| `dispatched` | CEO 已发出任务 | CEO 发送派工消息 |
| `accepted` | Agent 已确认接单 | Agent 回复 ACK |
| `running` | Agent 已开始执行 | Agent 开始工作 |
| `waiting_review` | Agent 已提交产出 | Agent 提交产出文件 |
| `done` | CEO 验收通过 | CEO 确认验收 |
| `failed_no_ack` | Agent 未确认接单 | 60 秒无 ACK |
| `failed_timeout` | Agent 超时无产出 | 超过 deadline 无产出 |

### 2.3 超时规则
- **ACK 超时**：60 秒无 ACK → `failed_no_ack`
- **进度超时**：10 分钟无进度更新 → 升级提醒
- **Deadline 超时**：超过 deadline → `failed_timeout`

---

## 3. 派工流程

### 3.1 标准流程
```
CEO 发出任务 (dispatched)
    ↓
Agent 回复 ACK (accepted) [60秒内]
    ↓
Agent 开始执行 (running)
    ↓
Agent 提交产出 (waiting_review)
    ↓
CEO 验收 (done)
```

### 3.2 异常流程
```
CEO 发出任务 (dispatched)
    ↓
60秒无 ACK → failed_no_ack → CEO 升级处理
    ↓
Agent 回复 ACK (accepted)
    ↓
10分钟无进度 → CEO 提醒
    ↓
超过 deadline → failed_timeout → CEO 升级处理
```

---

## 4. 派工消息格式

### 4.1 CEO → Agent 派工消息
```
[任务ID] [任务名称]
Owner: [Agent名称]
Deadline: [截止时间]
产出: [产出文件路径]
验收: [验收标准]

请回复 ACK 确认接单。
```

### 4.2 Agent → CEO ACK 消息
```
ACK: [任务ID] 已接单
预计完成: [时间]
```

### 4.3 Agent → CEO 进度更新
```
[任务ID] 进度: [百分比]%
当前: [当前状态]
```

### 4.4 Agent → CEO 产出提交
```
[任务ID] 已完成
产出: [产出文件路径]
等待验收
```

### 4.5 CEO → Agent 验收确认
```
[任务ID] 验收通过
状态: done
```

---

## 5. 状态同步机制

### 5.1 message_bus 状态记录
每个状态变更都必须写入 message_bus：
```json
{
  "message_id": "M[ID]-[action]",
  "task_id": "[任务ID]",
  "from_agent": "[发送方]",
  "to_agent": "[接收方]",
  "message_type": "status_update",
  "payload": "状态: [新状态]",
  "timestamp": "[ISO时间]"
}
```

### 5.2 状态查询
CEO 可随时查询任务状态：
```
任务ID: [任务ID]
当前状态: [状态]
最后更新: [时间]
产出路径: [路径]
```

---

## 6. 超时处理机制

### 6.1 ACK 超时 (60秒)
1. CEO 检查 message_bus 是否有 ACK
2. 无 ACK → 标记 `failed_no_ack`
3. CEO 决定：重试 或 换 Agent 或 升级

### 6.2 进度超时 (10分钟)
1. CEO 检查是否有进度更新
2. 无更新 → 发送提醒消息
3. 仍无响应 → 升级处理

### 6.3 Deadline 超时
1. CEO 检查是否超过 deadline
2. 超过 → 标记 `failed_timeout`
3. CEO 决定：延期 或 换 Agent 或 取消任务

---

## 7. 最小任务验证

### 7.1 验证任务
**任务**：memory_agent 读取一份已有报告，并把摘要写入 memory/2026-05-14.md

**要求**：
1. CEO 发出任务
2. memory_agent 必须回复 ACK
3. ACK 后才进入 running
4. 产出路径固定为 memory/2026-05-14.md
5. 完成后 memory_agent 回复 waiting_review
6. CEO 验收后回复 done
7. 全过程写入 message_bus 状态副本

### 7.2 验证标准
- [ ] CEO 发出派工消息
- [ ] memory_agent 60秒内回复 ACK
- [ ] memory_agent 执行任务
- [ ] memory_agent 提交产出
- [ ] CEO 验收通过
- [ ] message_bus 有完整状态记录

---

## 8. CEO 职责

### 8.1 必须做
1. 派工时明确所有要素
2. 等待 ACK 确认
3. 监督进度
4. 验收产出
5. 记录状态

### 8.2 禁止做
1. 亲自执行任务
2. 不等 ACK 就认为派工完成
3. 不监督进度
4. 不验收产出

---

## 9. Agent 职责

### 9.1 必须做
1. 收到任务后 60 秒内回复 ACK
2. 按时完成任务
3. 提交产出到指定路径
4. 等待 CEO 验收

### 9.2 禁止做
1. 不回复 ACK
2. 不按时完成
3. 不提交产出
4. 不等待验收就标记完成

---

## 10. 版本历史

- **v1.0** (2026-05-14)：初始版本，建立握手机制

---

**机制状态**：✅ 正式生效  
**立即验证**：memory_agent 最小任务  
**维护者**：千尘（CEO）