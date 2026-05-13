# Codex 加入 AI 公司组织模型 v1

**版本**：1.0  
**设计时间**：2026-05-14 07:40 GMT+8  
**设计者**：千尘（CEO）  
**状态**：设计阶段，不写代码

---

## 1. 定位与目标

### 1.1 核心定位
**Codex 是 AI 公司的工程外挂 / 总监察官 / 超级工程外援。**

### 1.2 角色边界
- ✅ **是**：工程外挂、总监察官、超级工程外援
- ❌ **不是**：CEO、普通子Agent、message_bus worker
- ❌ **不自动接单**：只在 CEO 授权后工作
- ❌ **不替代 CEO 决策**：只提建议，不拍板

### 1.3 建设目标
1. **目标1**：让千尘能真实调动子 Agent 干活（第二阶段）
2. **目标2**：让 Codex 加入 AI 公司，成为真实工程外挂（第一阶段）

### 1.4 执行顺序
**第一阶段**：先接 Codex（当前真实可用能力）  
**第二阶段**：再做子 Agent worker/ACK（当前 unavailable）

---

## 2. 组织关系

### 2.1 CEO（千尘）
**职责**：判断、拆解、授权、验收、记忆沉淀、用户汇报

**具体工作**：
1. **判断任务**：分析是否需要 Codex 审查
2. **创建任务单**：编写 Codex 任务单文件
3. **授权调用**：通过 Human-Tool/Codex Adapter 调用
4. **读取报告**：读取 Codex 输出的报告文件
5. **验收结果**：检查输出质量和边界
6. **写入 memory**：将摘要写入 memory/2026-05-14.md
7. **向用户汇报**：通过 Telegram 发送摘要和链接

### 2.2 Codex（工程外挂/总监察官）
**职责**：代码审查、补丁草稿、diff审查、本地验证建议、PR review、风险审查

**具体工作**：
1. **代码审查**：分析代码质量、安全漏洞、性能问题
2. **补丁草稿**：生成修复补丁（.patch 文件）
3. **diff 审查**：审查代码差异和变更
4. **本地验证建议**：提供测试建议、验证命令
5. **PR review**：审查 Pull Request
6. **风险审查**：识别潜在风险和问题

**工作模式**：
- 只读模式：不直接修改文件
- 补丁模式：只生成补丁，不直接应用
- 异步执行：不阻塞 CEO

### 2.3 Human-Tool / Codex Adapter
**职责**：当前阶段负责实际调用 Codex

**具体工作**：
1. **实际调用**：通过 API 或工具调用 Codex
2. **不通过 tools_agent worker**：当前 tools_agent 无 worker/ACK 机制
3. **不依赖 message_bus 自动接单**：手动触发调用
4. **记录日志**：记录调用过程和结果

### 2.4 子 Agent（未来）
**当前状态**：只作为职责角色和会议视角

**未来接入**：
- 在 worker/ACK 实现前，不作为真实执行体
- 未来可以通过 worker/ACK 接入执行闭环
- 当前只参与会议讨论，不参与实际执行

---

## 3. 3D 办公室展示

### 3.1 显示角色
**角色名称**：外部工程顾问 / Codex 工程外挂

**显示位置**：
- 3D 办公室独立区域
- 不显示为普通员工 Agent
- 标识为"外部顾问"或"工程外挂"

### 3.2 状态显示
| 状态 | 图标 | 颜色 | 含义 |
|------|------|------|------|
| `idle` | ⚪ | 白色 | 空闲 |
| `reviewing` | 🔍 | 蓝色 | 正在审查 |
| `patching` | 📝 | 黄色 | 正在生成补丁 |
| `cooldown` | ⏸️ | 灰色 | 额度耗尽 |

### 3.3 状态更新时机
- **开始任务**：更新为 `reviewing` 或 `patching`
- **任务完成**：更新为 `idle`
- **进入 cooldown**：更新为 `cooldown`
- **退出 cooldown**：更新为 `idle`

---

## 4. 任务系统接入

### 4.1 任务进入方式
**通过 Codex 任务单进入**

**任务单格式**：
```yaml
task_id: "CODEX-[YYYYMMDD]-[序号]"
task_type: "read-only|patch-only|local-verify|PR-review"
repo: "仓库路径"
target_files: ["文件路径列表"]
scope: "审查/修复范围描述"
forbidden: ["禁止操作列表"]
acceptance_criteria: ["验收标准列表"]
timeout: "超时时间（秒）"
cost_limit: "成本限制（美元）"
output_path: "输出文件路径"
report_path: "报告文件路径"
```

### 4.2 任务类型限制
**只允许以下类型**：
1. `read-only`：只读审查
2. `patch-only`：生成补丁草稿
3. `local-verify`：本地验证/测试建议
4. `PR-review`：审查 PR 或 diff

### 4.3 结果存储
**所有结果写入**：`qianchen-os-docs/codex-reports/`

**文件命名**：`YYYY-MM-DD-[任务类型]-[来源].md`

---

## 5. CEO 记忆接入

### 5.1 三段式交付（基于《CEO 重要消息交付协议 v1》）

**第一段：写入持久化报告文件**
- 文件路径：`qianchen-os-docs/codex-reports/`
- 文件格式：标准 Markdown
- 内容要求：完整分析、证据、建议、验证结果

**第二段：短消息通知 CEO 读取**
- 消息格式：`Codex 任务完成：[简短描述]，报告：[文件路径]`
- 消息长度：不超过 200 字符
- 必须包含：任务类型 + 文件路径

**第三段：CEO 写入当天 memory**
- 文件路径：`memory/YYYY-MM-DD.md`
- 内容要求：摘要 + 链接，不整篇复制
- 格式：`### [时间] [任务类型] 已读取\n**核心结论**：[结论]\n**报告路径**：[路径]`

### 5.2 状态同步
- **message_bus**：只做状态副本，不作为主交付
- **Telegram**：只发摘要和链接，不发送完整报告
- **状态类型**：`codex_review`、`codex_cooldown`、`idle`

---

## 6. Usage Limit 处理

### 6.1 Cooldown 机制
**触发条件**：
1. **usage limit**：API 调用达到限制
2. **连续失败**：任务连续失败 2 次
3. **超时**：任务执行超时
4. **越界尝试**：尝试执行禁止操作

**Cooldown 后行为**：
1. **停止重试**：不自动重试任务
2. **转 Fallback**：转 GPT/Human-Tool fallback
3. **记录状态**：记录到 memory 和 qianchen-os-docs
4. **通知 CEO**：通过短消息通知 CEO

### 6.2 Fallback 策略（基于《OpenClaw 现实执行模型 v1》）
**按任务类型路由**：
1. **代码执行/本地验证**：Codex > Human-Tool > GPT
2. **架构判断/方案设计**：GPT > Codex > Human-Tool
3. **账号/凭证/系统设置**：Human-Tool > GPT > Codex
4. **记忆沉淀/报告归档**：CEO > Human-Tool > GPT

### 6.3 记录要求
**memory 记录**：
```
### [时间] Codex 进入 cooldown
**原因**：[触发原因]
**预计恢复**：[恢复时间]
**Fallback 方案**：[GPT/Human-Tool]
```

**qianchen-os-docs 记录**：
- 文件路径：`qianchen-os-docs/reports/YYYY-MM-DD-codex-cooldown.md`
- 内容：触发原因、影响范围、fallback 方案、恢复计划

---

## 7. 与现有基线的关系

### 7.1 基于《OpenClaw 现实执行模型 v1》
- **真实可用能力**：Codex 是当前真实可用 fallback
- **执行优先级**：按任务类型路由，Codex 是代码执行/本地验证的首选
- **成本控制**：避免无边界使用，只用于关键任务

### 7.2 基于《CEO 重要消息交付协议 v1》
- **三段式交付**：文件 → 通知 → 记忆沉淀
- **验收标准**：必须报告文件、必须验证命令、必须写入 memory
- **状态同步**：message_bus 只做状态副本

### 7.3 基于《子 Agent 派工机制验证失败归档报告》
- **派工铁律**：没有 ACK 不算派工，没有 worker 不算可执行
- **当前状态**：子 Agent 派工能力 unavailable
- **执行顺序**：第一阶段先接 Codex，第二阶段再做子 Agent worker/ACK

---

## 8. 版本历史

- **v1.0** (2026-05-14)：初始设计，定义 Codex 在 AI 公司中的组织角色

---

**设计状态**：✅ 完成  
**下一步**：等待 CEO 审批  
**维护者**：千尘（CEO）