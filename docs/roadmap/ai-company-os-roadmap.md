# AI 公司操作系统路线图

> 版本：v1.0  
> 日期：2026-05-13  
> 状态：规划中

## 战略定位

**OpenClaw v2：AI 公司操作系统**

从对话型智能体升级为操作系统，核心要素：
- 工程能力 → Codex 代码臂
- 空间可视化 → 3D办公室 / Spatial OS
- 移动入口 → Telegram / 未来 App
- 长期心智 → V5 记忆系统

---

## Phase 1: Codex 只读接入 ✅

**状态：** 已完成  
**时间：** 2026-05-13  
**耗时：** 30分钟

### 完成内容
- [x] 注册4个白名单任务到 skill_registry
- [x] 配置 Codex Runner 只读模式
- [x] 实现 codex.check_v5 原型
- [x] Telegram摘要推送测试
- [x] 端到端验证全部4个任务

### 产出
- `ai-company/codex_readonly.sh` - 只读执行器
- `ai-company/codex_telegram_summary.py` - 摘要生成器
- `docs/architecture/codex-openclaw-integration-v1.md` - 架构文档

---

## Phase 2: 异步派工模型

**状态：** 设计完成，待实现  
**时间：** 待定  
**前置条件：** Phase 1 运行稳定

### 设计内容
- [x] 任务状态机（queued → running → waiting_review → done/failed/safe_mode）
- [x] 任务单结构
- [x] 派工协议
- [x] 回报协议
- [x] 安全规则

### 实现内容（待启动）
- [ ] 任务队列管理
- [ ] 执行层改造
- [ ] 回报机制
- [ ] 验收流程

### 产出
- `docs/architecture/async-dispatch-v1.md` - 设计文档

---

## Phase 3: 3D办公室HUD接入

**状态：** 规划中  
**时间：** 待定  
**前置条件：** Phase 2 完成

### 计划内容
- [ ] 定义HUD数据接口
- [ ] Agent状态实时推送
- [ ] 任务进度可视化组件
- [ ] 集成测试

### 目标
- 3D办公室显示实时任务状态
- Agent工作可视化
- 任务进度监控

---

## Phase 4: 写入能力解封

**状态：** 规划中  
**时间：** 待定  
**前置条件：** Phase 1-3 稳定运行 ≥ 7天

### 计划内容
- [ ] 解封 `codex.apply_diff`
- [ ] git 自动分支
- [ ] 变更预览
- [ ] 人工审批流程

### 安全措施
- 写入操作必须人工审批
- git 自动创建分支
- 变更预览确认
- 强制 code review

---

## Phase 5: 公共仓库沉淀

**状态：** 进行中  
**时间：** 2026-05-13  
**耗时：** 进行中

### 完成内容
- [x] 创建公共仓库结构
- [x] 定义沉淀规范
- [x] 清理敏感信息
- [x] 上传架构文档

### 产出
- `qianchen-os-docs/` - 公共文档仓库
- `README.md` - 使用规范

---

## 关键里程碑

| 里程碑 | 状态 | 日期 | 备注 |
|--------|------|------|------|
| Phase 1 完成 | ✅ | 2026-05-13 | Codex 只读接入 |
| 异步派工模型设计 | ✅ | 2026-05-13 | 设计基线 |
| 公共仓库创建 | ✅ | 2026-05-13 | 非敏感资料沉淀 |
| Phase 2 实现 | ⏳ | 待定 | 异步派工 |
| Phase 3 实现 | ⏳ | 待定 | 3D办公室HUD |
| Phase 4 实现 | ⏳ | 待定 | 写入能力 |

---

## 风险与依赖

### 技术风险
- Codex API 成本控制
- 异步任务队列稳定性
- 3D办公室性能

### 依赖项
- OpenAI Codex API 可用性
- 3D办公室开发进度
- Telegram Bot 稳定性

---

## 资源需求

### 人力资源
- coding_agent：代码实现
- tools_agent：工具集成
- qa_agent：测试验证
- ceo：架构设计、派工、验收

### 基础设施
- 本地服务器（已有）
- Codex API 配额
- 公共 Git 仓库

---

*路线图 — 2026-05-13*
