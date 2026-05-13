# generate_proposals 真实数据驱动 — GPT 实现任务

## 文件
`/home/administrator/.openclaw/workspace/ai-company/coding_agent_server.py`

## 当前问题
`generate_proposals` 返回固定模板方案，无真实数据支撑。

## 需要修改的函数
`def generate_proposals():` （约第 592 行开始）

## 输入数据（已有，直接读取）

```python
# task-ledger（任务列表）
from task_ledger import list_tasks as _list_tasks
tasks = _list_tasks()  # 返回 list[dict]，每个有 task_id, title, state, assignee, priority, started_at, progress

# blocks（阻塞列表）
from blocks import get_active_blocks
blocks = get_active_blocks()  # 返回 list[dict]，每个有 block_id, task_id, description, severity

# agents-state（agent 状态）
from coding_agent_server import load_agents_state  # 或直接读 agents-state.json
agents = load_agents_state()  # 返回 list[dict]，每个有 agentId, name, state, detail

# meeting topic
topic = request.args.get("topic", "讨论")

# meeting history（可选，前端传来）
history = request.args.get("history", "[]")
```

## 输出格式（每个方案必须包含）

```json
{
  "name": "方案A：清除3个阻塞释放下游",
  "core_idea": "集中处理 T-001/T-002/T-003 三个阻塞任务，释放 coding_agent 和 qa_agent 的下游工作流",
  "recommend_reason": "当前有3个阻塞任务阻塞了5个下游任务，清除后可释放2天工期",
  "pros": ["释放下游依赖", "减少上下文切换", "快速见效"],
  "cons": ["暂停新需求2天", "需要全员配合"],
  "cost": "低",
  "time": "1-2天",
  "impact": "5个下游任务可推进",
  "owner": "coding_agent",
  "priority": "high",
  "steps": [
    "分析3个阻塞任务（T-001: xxx, T-002: xxx, T-003: xxx）的根因",
    "制定修复方案并分配给构术/守衡",
    "24小时内完成修复并更新任务状态",
    "验证下游任务依赖是否已释放"
  ],
  "risks": [
    "阻塞根因比预期复杂，可能需要架构调整",
    "修复期间可能引入新 bug，需要回归测试"
  ],
  "acceptance_criteria": [
    "3个阻塞任务状态变为 done",
    "下游5个任务可正常开始",
    "无新增阻塞"
  ],
  "estimated_effort": "1-2天",
  "related_tasks": ["T-001", "T-002", "T-003"],
  "evidence": [
    "task-ledger: 3个阻塞任务阻塞了5个下游",
    "blocks: T-001 (描述), T-002 (描述), T-003 (描述)",
    "agents-state: coding_agent idle, qa_agent idle"
  ]
}
```

## 方案生成规则

### 规则1：如果有阻塞 → 生成"清除阻塞"方案
- steps 必须引用具体阻塞任务的 task_id 和 title
- owner 根据阻塞任务的 assignee 确定
- evidence 引用 blocks 数据

### 规则2：如果有 in_progress 任务 > 3 → 生成"收敛在制品"方案
- steps 必须列出具体进行中任务的 title
- 引用 task-ledger 中 state=in_progress 的任务
- evidence 引用 tasks 数据

### 规则3：如果有 agent 异常 → 生成"恢复异常"方案
- 引用 agents-state 中 state=error 的 agent
- steps 针对具体异常原因

### 规则4：始终生成一个"推进主线"兜底方案
- 基于 meeting topic 生成
- steps 描述下一步行动

### 禁止
- 禁止出现"修改、验证、优化"这种无上下文空任务
- 每个 step 必须有具体对象（任务ID、agent名、文件名等）
- 每个 risk 必须是具体风险（不是"可能有风险"）

## 验收标准
1. ✅ 每个方案 ≥ 3 个 steps
2. ✅ 每个方案 ≥ 2 个 risks
3. ✅ 每个方案 ≥ 2 个 acceptance_criteria
4. ✅ 每个方案有 owner 和 priority
5. ✅ 每个方案有 recommend_reason（为什么推荐）
6. ✅ 无"修改/验证/优化"等空任务
7. ✅ 引用至少一项真实数据（task_id/block_id/agent名）

## 注意
- 前端 ProposalReview.tsx 已兼容 name/core_idea/pros/cons/cost/time/impact/tasks/owner/priority/risks/acceptance_criteria
- 不要改 /api/proposals/approve 或 /api/bridge/approved-to-board
- 不要改会议 SSE
- 不要接 Telegram
- 只改 generate_proposals 函数
