# Skill 使用审计报告（修正版）

> 日期：2026-05-13 06:15  
> 审计者：千尘 🌌  
> 状态：只读检查，未修改文件  
> 修正：基于雨陌反馈修正结论

## 一、审计发现

### 1. 已安装 Skills

**总数：** 32个

**目录：** `~/.openclaw/workspace/skills/`

**完整列表：**
- agent-creator
- agent-team-orchestration
- architecture-patterns
- code-architect
- code-reviewer
- code-simplifier
- code-understand
- ddg-search
- debug-pro
- deep-research-pro
- **dispatching-parallel-agents** ← 派单 skill
- explore
- frontend-design
- hook-development
- memory-audit
- memory-intake
- memory-model-builder
- openclaw-code-guard
- openclaw-health-doctor
- pr-test-analyzer
- proactivity
- security-review
- self-improving-agent
- silent-failure-hunter
- skill-security-auditor
- social-media-scheduler
- test-driven-development
- type-design-analyzer
- understand-image-minimax
- verification
- workspace-guard
- writing-hookify-rules

### 2. 已注册到 skill_registry

**总数：** 36个（包括4个 Codex 白名单任务）

**文件：** `ai-company/org/skill_registry.yaml`

**注册状态：** ✅ 所有已安装的 skills 都已注册

### 3. 派单 Skill 详情

**名称：** `dispatching-parallel-agents`

**路径：** `~/.openclaw/workspace/skills/dispatching-parallel-agents/SKILL.md`

**功能：**
- 当面临2+个独立任务时，可以并行派发给多个Agent
- 每个Agent负责一个独立的问题域
- 适合：多个测试失败、多个子系统故障、多个不相关bug

**触发条件：**
- 多个失败/任务
- 这些失败是独立的（修复一个不会影响另一个）
- 可以并行工作（无共享状态）
- 不需要理解完整系统状态

**不适用场景：**
- 失败是相关的
- 需要理解完整系统状态
- 探索性调试
- 共享状态（编辑相同文件、使用相同资源）

---

## 二、最近任务派单使用情况

### 1. Codex 接入

**是否使用派单 skill：** ❌ 否

**原因分析：**
- 任务是串行的（先注册，再测试，再验证）
- 每个步骤依赖前一个步骤的结果
- 不符合并行条件

**正确做法：** 串行执行，无需派单

### 2. 异步派工模型设计

**是否使用派单 skill：** ❌ 否

**原因分析：**
- 这是一个设计任务，不是执行任务
- 需要完整的上下文和连贯的思考
- 不适合拆分给多个Agent

**正确做法：** 单一Agent完成设计

### 3. V5 健康检查

**是否使用派单 skill：** ❌ 否

**原因分析：**
- 单一检查任务
- 需要理解完整的V5系统状态
- 不是多个独立问题

**正确做法：** 单一Agent完成检查

### 4. 3D办公室事故复盘

**是否使用派单 skill：** ❌ 否

**原因分析：**
- 复盘需要完整的上下文
- 需要理解事故的完整时间线
- 不是多个独立问题

**正确做法：** 单一Agent完成复盘

---

## 三、派单 Skill 使用历史

**总派单次数：** 2次

**最近派单记录：**
1. `trace-smoke-001` - 链路测试：执行实现
2. `trace-ui-001` - 链路UI验收：执行实现

**派单频率：** 极低（历史总使用次数仅2次）

---

## 四、问题诊断（修正版）

### 1. 派单 Skill 未被使用的原因

**主要原因：**
1. **任务类型不匹配** - 最近的任务都是单一任务或串行任务
2. **未识别并行机会** - 没有遇到需要并行处理的场景
3. **触发条件未触发** - 没有遇到2+个独立任务的情况

**不是原因：**
- ✅ 已安装
- ✅ 已注册到 skill_registry
- ✅ 功能描述清晰
- ✅ 触发条件明确

### 2. 其他 Skills 使用情况

**已使用 skills：**
- 无明确记录（缺少 skill 调用日志）

**可能未使用 skills：**
- 大部分 skills 可能未被触发
- 需要检查每个 skill 的触发条件

### 3. 关键断点

**缺少 skill 调用日志：**
- 无法追溯 skill 是否被识别
- 无法追溯 skill 是否被调用
- 无法追溯为什么没有调用
- 无法验证 skill 调度能力是否正常

---

## 五、修正结论

### 1. 派单 Skill 状态

**结论：** 派单 skill 可用，但不解决异步不阻塞问题。

**说明：**
- 派单 skill 解决的是"并行派单"（多个独立任务同时执行）
- 异步不阻塞是另一个问题（ceo 派工后不等待，继续对话）
- 两者是不同维度的问题

### 2. Skill 调度能力

**结论：** OpenClaw 的 skill 调度能力仍未充分验证。

**原因：**
1. 缺少 skill 调用日志
2. 无法确认 skill 是否被正确识别
3. 无法确认 skill 是否被正确调用
4. 无法确认为什么某些 skill 未被调用

### 3. 下一步建议

**不要继续造一个完全新的派工系统。**

**应该：**
1. 复用现有 `dispatching-parallel-agents` 处理多独立任务并行
2. 新增轻量异步任务队列，解决 ceo 长任务不阻塞
3. 新增 skill 调用日志，记录每次 skill 是否被识别、是否调用、为什么没调用
4. ceo 每次遇到任务先做 skill 匹配检查

---

## 六、建议

### 1. 派单 Skill 接入 ceo 默认工作流

**建议做法：**
1. 在 ceo 的决策流程中添加"并行检查"
2. 当遇到多个任务时，先判断是否独立
3. 如果独立且可并行，使用派单 skill

**触发条件：**
- 用户提出多个不相关的任务
- 系统出现多个独立故障
- 需要同时检查多个方面

### 2. Skill 调用日志

**建议做法：**
1. 记录每次 skill 调用
2. 记录：skill 名称、是否被识别、是否被调用、为什么没调用
3. 定期审计 skills 使用情况

### 3. 轻量异步任务队列

**建议做法：**
1. 复用现有派单 skill 处理并行
2. 新增轻量队列处理异步
3. ceo 派工后不等待，继续对话

---

## 七、总结

**关键发现：**
1. 派单 skill 已安装并注册，但最近未使用
2. 最近的任务类型不匹配派单 skill 的触发条件
3. 派单 skill 功能正常，随时可用

**修正结论：**
1. 派单 skill 不是坏的
2. 最近未用有合理原因
3. 但 OpenClaw 的 skill 调度能力仍未充分验证
4. 缺少 skill 调用日志是当前断点
5. 现有派单 skill 解决的是"并行派单"，不是"异步不阻塞"

**下一步：**
1. 不要继续造一个完全新的派工系统
2. 复用 dispatching-parallel-agents 处理多独立任务并行
3. 新增轻量异步任务队列，解决 ceo 长任务不阻塞
4. 新增 skill 调用日志
5. ceo 每次遇到任务先做 skill 匹配检查

---

*审计报告（修正版） — 2026-05-13 06:15*
