# Codex 政委 Phase 2 —— 会议接入方案

**任务 ID**：T0514085200-tools  
**项目**：CODEX-COMMISSAR-PHASE1  
**优先级**：P1  
**日期**：2026-05-14  
**作者**：tools_agent（生钱官）  
**范围**：只出文档，不改代码  
**前置文件**：
- `qianchen-os-docs/docs/architecture/codex-commissar-organization-design-v1.md`
- `qianchen-os-docs/docs/standards/codex-commissar-operating-standard-v1.md`
- `qianchen-os-docs/reports/2026-05-14-codex-commissar-phase1-execution-codex.md`

---

## 一、Phase 2 定位

### 1.1 与 Phase 1 的关系

| 维度 | Phase 1（已完成） | Phase 2（本文档） |
|------|-------------------|-------------------|
| 目标 | 制度建立 | 会议流程接入 |
| 产出 | 组织设计 + 运行标准 | 触发机制 + 会议规范 + UI 需求 |
| 代码变更 | 无 | 无（仅文档） |
| Codex 介入方式 | 文件审查 | 会议审查 + 口头/书面结论 |

### 1.2 Phase 2 核心目标

> 在 3D 办公室会议系统中加入 Codex 审查环节，实现「CEO 有疑问 → 自动召唤 Codex → 会议区展示审查结论 → CEO 拍板」。

### 1.3 不做的事

- ❌ 不改代码（本阶段纯文档）
- ❌ 不实现 Hub 端点
- ❌ 不修改 3D 办公室前端
- ❌ 不接入 message_bus 自动接单

---

## 二、现有系统状态

### 2.1 3D 办公室会议系统

**现状**：3D 办公室（`3d-office/`）已存在 Next.js 应用，包含：
- Agent 状态展示（`/api/office`）
- 任务列表（`/api/tasks`）
- 活动日志（`/api/activity`）
- 任务调度（Hub API）

**会议区**：当前 3D 办公室有 Agent 协作区域，但无专门的"Codex 审查"会议形态。

### 2.2 现有 Agent 注册表

**文件**：`ai-company/agent-registry.json`

当前 Agent：`ceo`（千尘）、`pm_agent`、`coding_agent`、`qa_agent`、`tools_agent`、`memory_agent`、`ux_agent`

**缺少**：`codex_commissar` 未注册为 Agent。

### 2.3 现有 Hub API

**状态同步**：`/push` —— 可推 Agent 状态  
**Agent 列表**：`/api/office` —— 返回已注册 Agent 状态  
**活动记录**：`/api/activity` —— 可记录审查活动

---

## 三、会议流程设计

### 3.1 标准会议流程（含 Codex 环节）

依据 `codex-commissar-operating-standard-v1.md` 第 5 节，完整会议流程如下：

```
┌─────────────────────────────────────────────────┐
│              3D 办公室标准会议流程               │
├─────────────────────────────────────────────────┤
│                                                 │
│  1. CEO 开场 ──── 说明目标、议题               │
│  2. pm_agent ──── 需求与优先级                 │
│  3. coding_agent ─ 实现路径                    │
│  4. qa_agent ──── 验收与风险                   │
│  5. tools_agent ── 工具链与执行链路            │
│  6. ux_agent ──── 体验影响                     │
│  7. memory_agent ─ 历史经验                    │
│  8. 子 Agent 互相质询 ── 交叉验证              │
│  9. 形成候选方案 ──── 汇总意见                │
│                                                 │
│  ══════════════════════════════════════════     │
│  10. Codex 政委审查 ← ★ 新增环节 ★           │
│  ══════════════════════════════════════════     │
│                                                 │
│  11. CEO 最终拍板                              │
│  12. CEO 创建任务并派工                        │
│  13. 子 Agent 执行                             │
│  14. Codex 抽查或终审（按需）                  │
│  15. CEO 汇报投资人                            │
│                                                 │
└─────────────────────────────────────────────────┘
```

### 3.2 Codex 审查环节详细流程

```
CEO 判断是否需要 Codex 审查
         │
    ┌────┴────┐
    │         │
   低风险    高风险/关键
    │         │
 跳过 Codex  召唤 Codex
    │         │
    │    ┌────┴───────────────────────┐
    │    │  CEO 触发 POST /api/codex/summon
    │    │  参数：task_id, reason, mode
    │    │                            │
    │    │  Hub 写入 codex-reviews.jsonl
    │    │  Hub push codex_commissar status=reviewing
    │    │                            │
    │    │  3D 办公室会议区显示        │
    │    │  「Codex 审查」卡片         │
    │    │                            │
    │    │  Codex 执行审查（异步）     │
    │    │                            │
    │    │  审查完成 → 写入报告文件    │
    │    │  → Hub push codex_commissar status=idle
    │    │  → 会议区卡片更新为结论    │
    │    │                            │
    │    └────────────────────────────┘
    │         │
    ▼         ▼
 CEO 拍板（基于所有输入 + Codex 结论）
```

### 3.3 低风险会议跳过规则

低风险会议可跳过 Codex 审查，但**会议记录必须注明**：
- 跳过原因（如"低风险 UI 文案调整"）
- 决策依据（如"qa_agent 初验通过"）

判断标准参考 `codex-commissar-operating-standard-v1.md` 第 3.3 节"不应调用 Codex"条件。

---

## 四、触发机制设计

### 4.1 Codex 召唤端点

**端点**：`POST /api/codex/summon`

**请求参数**：

```json
{
  "task_id": "T0514085200-001",          // 任务 ID
  "reason": "跨仓库改动审查",             // 召唤原因
  "mode": "read-only",                    // 审查模式
  "scope": "审查范围描述",                // 审查范围
  "target_files": [                       // 目标文件列表（可选）
    "qianchen-os-docs/docs/architecture/*.md"
  ],
  "timeout_sec": 300,                     // 超时秒数
  "cost_limit_usd": 2.0,                  // 成本上限
  "requested_by": "ceo"                   // 召唤者
}
```

**mode 枚举值**：

| mode | 含义 | 使用场景 |
|------|------|----------|
| `read-only` | 只读审查 | 方案审查、风险评估（默认） |
| `patch` | 生成补丁 | 高风险修复建议（需 CEO 批准） |
| `final-review` | 终审 | 执行结果抽查、PR 合并前审查 |

**响应**：

```json
{
  "status": "accepted",
  "review_id": "CR-20260514-001",
  "estimated_wait_sec": 60,
  "codex_status": "reviewing"
}
```

### 4.2 Codex 状态查询端点

**端点**：`GET /api/codex/status`

**响应**：

```json
{
  "agent_id": "codex_commissar",
  "status": "reviewing",                // idle | reviewing | patching | cooldown
  "current_review_id": "CR-20260514-001",
  "current_review_task_id": "T0514085200-001",
  "cooldown_remaining_sec": 0,
  "last_completed_at": "2026-05-14T08:30:00+08:00",
  "today_review_count": 3,
  "today_cost_usd": 1.85
}
```

### 4.3 审查结果查询端点

**端点**：`GET /api/codex/review/{review_id}`

**响应**：

```json
{
  "review_id": "CR-20260514-001",
  "status": "completed",               // pending | running | completed | failed | timeout
  "mode": "read-only",
  "conclusion": "CONDITIONAL_PASS",     // PASS | CONDITIONAL_PASS | FAIL
  "summary": "方案整体可行，3 个风险点需关注",
  "report_path": "qianchen-os-docs/reports/2026-05-14-codex-review-001.md",
  "completed_at": "2026-05-14T08:35:00+08:00",
  "cost_usd": 0.62,
  "risk_items": [
    {"level": "HIGH", "desc": "异步消息可能丢失"},
    {"level": "MED", "desc": "超时重试策略未定义"}
  ]
}
```

### 4.4 触发条件自动判断（参考）

Phase 2 仅支持**手动触发**（CEO 或子 Agent 主动召唤）。自动判断留待 Phase 4。

以下为参考判断逻辑，供 Phase 4 设计时使用：

```
如果 满足"必须调用 Codex"任一条件 → 自动召唤
如果 满足"可以调用 Codex"条件 → 提示 CEO，由 CEO 决定
否则 → 跳过，不提示
```

条件定义见 `codex-commissar-operating-standard-v1.md` 第 3 节。

---

## 五、3D 办公室 UI 变更需求

### 5.1 会议区 Codex 审查卡片

**触发**：Codex 审查开始后 3 秒内出现  
**位置**：3D 办公室会议区，区别于普通 Agent 会议卡片  
**设计要求**：

```
┌──────────────────────────────────────────┐
│  🔍 Codex 政委审查中...                   │
│                                          │
│  任务：T0514085200-001                   │
│  模式：只读审查                          │
│  原因：跨仓库改动审查                    │
│  状态：⏳ 审查中...                      │
│  耗时：00:01:23                          │
│                                          │
│  [取消审查]                              │
└──────────────────────────────────────────┘
```

**审查完成后的卡片**：

```
┌──────────────────────────────────────────┐
│  ✅ Codex 政委审查完成                    │
│                                          │
│  任务：T0514085200-001                   │
│  结论：CONDITIONAL PASS                  │
│  摘要：方案可行，3 个风险点需关注        │
│  耗时：00:02:15                          │
│  成本：$0.62                             │
│                                          │
│  [查看完整报告]                          │
└──────────────────────────────────────────┘
```

### 5.2 Star Office 会议面板

**变更**：会议面板新增 Codex 审查状态栏  
**显示内容**：Codex 审查状态、结论、报告路径  
**交互**：点击可查看完整报告或取消审查

### 5.3 Agent 状态列表

**变更**：`/api/office` 返回的 Agent 列表新增 `codex_commissar`

**codex_commissar 状态定义**：

| 状态 | 图标 | 颜色 | 含义 |
|------|------|------|------|
| `idle` | ⚪ | 白色/灰色 | 空闲，可接受审查任务 |
| `reviewing` | 🔍 | 蓝色 | 正在审查 |
| `patching` | 📝 | 黄色 | 正在生成补丁 |
| `cooldown` | ⏸️ | 橙色 | 额度耗尽或连续失败 |

---

## 六、数据流与状态管理

### 6.1 完整数据流

```
┌─────────────────────────────────────────────────────────────────┐
│                     Phase 2 数据流                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CEO/子Agent                                                    │
│    │                                                            │
│    │ POST /api/codex/summon                                     │
│    ▼                                                            │
│  Hub API                                                        │
│    ├── 写入 codex-reviews.jsonl（任务记录）                     │
│    ├── 推送 codex_commissar status=reviewing                    │
│    ├── 通知 3D 办公室（WebSocket/SSE push）                     │
│    └── 触发 Codex CLI（异步执行）                               │
│                                                                 │
│  Codex CLI（异步）                                              │
│    │                                                            │
│    │ 执行审查                                                   │
│    ├── 写入报告文件                                             │
│    │   qianchen-os-docs/reports/YYYY-MM-DD-codex-review-XXX.md  │
│    └── 写入补丁文件（如 mode=patch）                            │
│        qianchen-os-docs/patches/YYYY-MM-DD-fix-XXX.patch        │
│                                                                 │
│  Hub API（结果回调）                                            │
│    ├── 更新 codex-reviews.jsonl（标记完成）                     │
│    ├── 推送 codex_commissar status=idle                         │
│    └── 通知 3D 办公室更新审查卡片为完成状态                     │
│                                                                 │
│  3D 办公室                                                       │
│    ├── 会议区显示/更新审查卡片                                   │
│    ├── 活动日志记录审查活动                                     │
│    └── 提供"查看报告"跳转                                       │
│                                                                 │
│  CEO（读取结果）                                                 │
│    ├── GET /api/codex/review/{id} 或读取报告文件                 │
│    ├── 判断结论（PASS/CONDITIONAL_PASS/FAIL）                   │
│    ├── 拍板决策                                                 │
│    └── 派工执行                                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 审查记录文件

**文件**：`codex-reviews.jsonl`  
**位置**：`ai-company/`  
**格式**：每行一条 JSON 记录

```json
{
  "review_id": "CR-20260514-001",
  "task_id": "T0514085200-001",
  "reason": "跨仓库改动审查",
  "mode": "read-only",
  "scope": "审查范围描述",
  "target_files": ["path/to/file.md"],
  "requested_by": "ceo",
  "requested_at": "2026-05-14T08:25:00+08:00",
  "status": "completed",
  "conclusion": "CONDITIONAL_PASS",
  "summary": "方案可行，3 个风险点需关注",
  "report_path": "qianchen-os-docs/reports/2026-05-14-codex-review-001.md",
  "completed_at": "2026-05-14T08:27:15+08:00",
  "duration_sec": 135,
  "cost_usd": 0.62
}
```

### 6.3 Agent 注册表更新

**文件**：`ai-company/agent-registry.json`

**需新增**：

```json
{
  "codex_commissar": {
    "name": "Codex 政委",
    "role": "commissar",
    "description": "总监察官 / 特级工程顾问",
    "status": "idle",
    "modes": ["read-only", "patch", "final-review"],
    "internal_only": false,
    "cost_class": "high",
    "auto_assign": false
  }
}
```

---

## 七、审查报告规范

### 7.1 报告文件路径

```
qianchen-os-docs/reports/YYYY-MM-DD-codex-review-[序号].md
```

### 7.2 报告文件模板

```markdown
# Codex 审查报告

**审查 ID**：CR-YYYYMMDD-XXX
**任务 ID**：TXXXXXXXXX-XXX
**模式**：read-only / patch / final-review
**审查时间**：YYYY-MM-DD HH:MM
**耗时**：MM:SS
**成本**：$X.XX

## 1. 审查范围

[被审查的文件/方案/代码范围]

## 2. 审查结论

**结论**：PASS / CONDITIONAL PASS / FAIL
**建议**：[CEO 是否可执行此方案]

## 3. 风险清单

| 级别 | 问题 | 建议措施 |
|------|------|----------|
| HIGH | [问题描述] | [建议] |
| MED | [问题描述] | [建议] |
| LOW | [问题描述] | [建议] |

## 4. 详细分析

[技术分析、代码审查、方案评估的详细内容]

## 5. 补丁建议（如适用）

[补丁 diff 或修复建议]

## 6. 验证建议

[测试命令、验证步骤]

## 7. 附件

- [相关文件路径]
```

### 7.3 交付链路

```
Codex 写报告文件（主事实源）
  ↓
Hub push codex_commissar status=idle（状态同步）
  ↓
会议区卡片更新为结论（可视化）
  ↓
CEO 读取报告
  ↓
CEO 判断是否采纳
  ↓
CEO 派工执行
  ↓
memory_agent 写入 memory 摘要
  ↓
qa_agent 验收三处证据（报告文件 + message_bus + memory）
```

---

## 八、成本控制

### 8.1 单次审查成本上限

| 模式 | 默认上限 | 说明 |
|------|----------|------|
| `read-only` | $2.00 | 只读审查，通常 $0.5-1.5 |
| `patch` | $5.00 | 补丁生成，通常 $1.0-3.0 |
| `final-review` | $3.00 | 终审，通常 $1.0-2.0 |

### 8.2 每日审查上限

| 类型 | 上限 | 说明 |
|------|------|------|
| 每日审查次数 | 10 次 | 防止过度调用 |
| 每日成本上限 | $15.00 | 超限进入 cooldown |
| 单次超时 | 300 秒 | 超时自动取消 |

### 8.3 Cooldown 机制

**触发条件**：
1. 每日成本超限
2. 连续失败 2 次
3. 审查超时
4. 尝试越界操作

**Cooldown 行为**：
1. 停止接受新审查任务
2. 转 GPT/Human-Tool Fallback
3. 记录到 `codex-reviews.jsonl` 和 `memory/YYYY-MM-DD.md`
4. 通知 CEO

---

## 九、与其他 Agent 的协作

### 9.1 CEO 召唤 Codex 的场景

| 场景 | 召唤者 | 审查模式 |
|------|--------|----------|
| 会议上形成候选方案 | CEO | `read-only` |
| 方案派工前终审 | CEO | `read-only` 或 `final-review` |
| 执行结果抽查 | CEO | `final-review` |
| 高风险修复 | CEO | `patch` |
| 子 Agent 执行失败 | CEO 或 coding_agent | `read-only` 或 `patch` |

### 9.2 与 qa_agent 的协作

```
qa_agent 初验完成
  ↓
CEO 判断是否需要 Codex 终审
  ├─ 低风险 → CEO 直接验收
  └─ 高风险 → 召唤 Codex final-review
       ↓
  Codex 终审结论：PASS / CONDITIONAL PASS / FAIL
       ↓
  CEO 根据结论决策
```

### 9.3 与 coding_agent 的协作

```
coding_agent 提交代码变更
  ↓
CEO 判断变更风险
  ├─ 低风险 → qa_agent 验收即可
  └─ 高风险 → 召唤 Codex read-only 审查
       ↓
  Codex 审查结论
       ├─ PASS → qa_agent 验收
       ├─ CONDITIONAL PASS → CEO 决定是否接受风险
       └─ FAIL → coding_agent 修改后重新提交
```

---

## 十、验收标准

### 10.1 文档验收（本阶段）

- [x] 本文档存在且可读
- [x] 包含触发机制设计
- [x] 包含会议流程设计
- [x] 包含 UI 变更需求
- [x] 包含数据流设计
- [x] 包含审查报告规范
- [x] 包含成本控制策略

### 10.2 实现阶段验收（Phase 2 实现时）

- [ ] `codex_commissar` 注册到 `agent-registry.json`
- [ ] `POST /api/codex/summon` 端点可用
- [ ] `GET /api/codex/status` 端点可用
- [ ] `GET /api/codex/review/{id}` 端点可用
- [ ] 3D 办公室会议区显示 Codex 审查卡片
- [ ] 审查完成后卡片更新为结论状态
- [ ] `codex-reviews.jsonl` 正确记录审查过程
- [ ] 审查报告文件按规范生成
- [ ] 审查完成后 3 秒内会议区可见卡片
- [ ] 活动日志记录审查活动

---

## 十一、风险与缓解

| 风险 | 级别 | 缓解措施 |
|------|------|----------|
| Codex API 成本过高 | HIGH | 严格成本上限 + cooldown 机制 + DeepSeek 代班 |
| 审查延迟阻塞会议 | MED | 异步执行，CEO 不阻塞等待 |
| 3D 办公室 WebSocket 推送失败 | MED | 回退到轮询模式 |
| 审查结果格式不一致 | LOW | 强制使用报告模板 |
| Codex 连续失败 | MED | 连续失败 2 次自动 cooldown + Fallback |

---

## 十二、后续阶段预告

| 阶段 | 目标 | 前置条件 |
|------|------|----------|
| Phase 2 实现 | 编码实现本方案 | 本文档审批通过 |
| Phase 3 | 执行监督（抽查机制） | Phase 2 稳定运行 |
| Phase 4 | 自动路由（自动判断是否调用 Codex） | Phase 3 完成 |

---

## 附录 A：文件依赖清单

本方案涉及以下文件：

| 文件 | 用途 | 变更类型 |
|------|------|----------|
| `ai-company/agent-registry.json` | 注册 codex_commissar | 新增条目 |
| `ai-company/codex-reviews.jsonl` | 审查记录 | 新建文件 |
| `qianchen-os-docs/reports/` | 审查报告输出目录 | 已有 |
| `qianchen-os-docs/patches/` | 补丁输出目录 | 已有 |
| `3d-office` 相关组件 | 会议区 Codex 卡片 | UI 变更 |
| Hub API 路由 | 新增 3 个端点 | 后端变更 |

---

**文档状态**：✅ 完成  
**下一步**：CEO 审批本文档 → 调度 coding_agent 实现 → qa_agent 验收  
**维护者**：tools_agent（生钱官）  

---

AGENT_DONE task_id=T0514085200-tools artifact=qianchen-os-docs/reports/2026-05-14-codex-commissar-phase2-meeting-plan-tools.md
