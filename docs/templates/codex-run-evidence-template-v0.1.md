# Codex 运行证据模板 v0.1

**版本**：0.1  
**设计时间**：2026-05-14 08:25 GMT+8  
**设计者**：千尘（CEO）  
**用途**：标准化 codex-runs/<task_id>/ 证据目录  
**状态**：设计阶段，不写代码

---

## 1. 目的

确保以后每个 Codex 任务都有原始调用证据，区分真实执行和人工整理。

---

## 2. 目录结构

```
qianchen-os-docs/codex-runs/<task_id>/
├── task.yaml              # 任务单文件
├── command.txt            # 原始调用命令
├── stdout.txt             # 标准输出
├── stderr.txt             # 标准错误
├── exit-code.txt          # 退出代码
├── last-message.md        # 最后消息
└── run-metadata.json      # 运行元数据
```

---

## 3. 文件格式定义

### 3.1 task.yaml
**用途**：记录本次运行使用的任务单

**格式**：直接复制原始任务单文件

**示例**：
```yaml
task_id: "CODEX-20260514-001"
task_type: "read-only"
repo: "/home/administrator/.openclaw/workspace/ai-company"
target_files:
  - "coding_agent_server.py"
  - "qianchen-os-docs/docs/specs/generate-proposals-v1.md"
scope: "只读审查 generate_proposals 函数是否符合规格"
forbidden:
  - "git push"
  - "修改 openclaw.json"
acceptance_criteria:
  - "报告包含对比分析"
timeout: 300
cost_limit: 0.30
report_path: "qianchen-os-docs/codex-reports/2026-05-14-read-only-generate-proposals.md"
```

### 3.2 command.txt
**用途**：记录实际执行的 Codex CLI 命令

**格式**：纯文本，完整命令行

**示例**：
```
codex exec --prompt "请审查 ai-company/coding_agent_server.py 中的 generate_proposals 函数是否符合 qianchen-os-docs/docs/specs/generate-proposals-v1.md 规格要求" --full-auto
```

**要求**：
- 必须是实际执行的命令
- 包含所有参数
- 不得事后编造

### 3.3 stdout.txt
**用途**：记录 Codex CLI 的标准输出

**格式**：纯文本，原始输出

**示例**：
```
Codex: 开始审查 generate_proposals 函数...
Codex: 读取 coding_agent_server.py (586-935行)
Codex: 读取 generate-proposals-v1.md
Codex: 分析输出结构...
Codex: 生成报告...
Codex: 完成，报告已写入 qianchen-os-docs/codex-reports/2026-05-14-read-only-generate-proposals.md
```

**要求**：
- 保存完整输出
- 不得截断或修改
- 包含时间戳（如有）

### 3.4 stderr.txt
**用途**：记录 Codex CLI 的标准错误

**格式**：纯文本，原始输出

**示例**：
```
Warning: API rate limit approaching
Error: Connection timeout after 30s
```

**要求**：
- 保存完整错误信息
- 即使为空也要创建文件
- 包含时间戳（如有）

### 3.5 exit-code.txt
**用途**：记录 Codex CLI 的退出代码

**格式**：纯文本，单行数字

**示例**：
```
0
```

**常见代码**：
- `0`：成功
- `1`：一般错误
- `2`：误用 shell 命令
- `126`：命令不可执行
- `127`：命令未找到
- `130`：被 Ctrl+C 终止

**要求**：
- 必须是实际退出代码
- 不得事后修改

### 3.6 last-message.md
**用途**：记录 Codex 的最后消息或最终输出摘要

**格式**：Markdown 文本

**示例**：
```markdown
# Codex 最终消息

**时间**：2026-05-14 08:30 GMT+8
**状态**：success

## 审查结论
generate_proposals 函数基本符合规格，符合度 85%。

## 关键发现
1. 输出结构完全符合规格
2. 核心字段齐全
3. 有 3 个改进点

## 报告路径
qianchen-os-docs/codex-reports/2026-05-14-read-only-generate-proposals.md
```

**要求**：
- 包含关键结论
- 包含报告路径
- 不得与 stdout 完全重复

### 3.7 run-metadata.json
**用途**：记录运行元数据，用于验证和统计

**格式**：JSON

**必须字段**：
```json
{
  "task_id": "CODEX-20260514-001",
  "task_type": "read-only",
  "repo": "/home/administrator/.openclaw/workspace/ai-company",
  "executor": "Human-Tool",
  "started_at": "2026-05-14T08:25:00Z",
  "finished_at": "2026-05-14T08:30:00Z",
  "codex_cli_version": "1.0.0",
  "sandbox": "read-only",
  "approval_mode": "never",
  "exit_code": 0,
  "usage_limit_detected": false,
  "boundary_violation_detected": false,
  "report_path": "qianchen-os-docs/codex-reports/2026-05-14-read-only-generate-proposals.md",
  "evidence_status": "actual_execution"
}
```

**字段说明**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| task_id | string | ✅ | 任务ID，与 task.yaml 一致 |
| task_type | string | ✅ | 任务类型：read-only/patch-only/local-verify/PR-review |
| repo | string | ✅ | 仓库路径 |
| executor | string | ✅ | 执行者：Human-Tool/Codex-Adapter/OpenClaw |
| started_at | string | ✅ | 开始时间 ISO-8601 |
| finished_at | string | ✅ | 结束时间 ISO-8601 |
| codex_cli_version | string | ✅ | Codex CLI 版本 |
| sandbox | string | ✅ | 沙箱模式：read-only（只读）或 workspace-write（可写） |
| approval_mode | string | ✅ | 审批模式：auto/manual |
| exit_code | number | ✅ | 退出代码 |
| usage_limit_detected | boolean | ✅ | 是否检测到 usage limit |
| boundary_violation_detected | boolean | ✅ | 是否检测到越界尝试 |
| report_path | string | ✅ | 报告文件路径 |
| evidence_status | string | ✅ | 证据状态：actual_execution/execution_unverified |

**evidence_status 取值**：
- `actual_execution`：有完整运行证据，Codex 真实执行
- `execution_unverified`：缺少运行证据，不能证明 Codex 真实执行

---

## 4. 证据验证规则

### 4.1 必须满足（全部通过）
1. ✅ **codex-runs 目录存在**：`qianchen-os-docs/codex-runs/<task_id>/`
2. ✅ **7 个文件齐全**：task.yaml、command.txt、stdout.txt、stderr.txt、exit-code.txt、last-message.md、run-metadata.json
3. ✅ **run-metadata.json 格式正确**：包含所有必填字段
4. ✅ **evidence_status 标注正确**：actual_execution 或 execution_unverified
5. ✅ **exit_code 一致**：exit-code.txt 与 run-metadata.json 一致

### 4.2 铁律
1. **没有 codex-runs 目录，不算 actual_execution**
2. **如果 Codex 没有真实运行，必须标记 execution_unverified**
3. **codex-reports 是整理报告，不是原始证据**
4. **memory 写入时必须标注 actual_execution 或 execution_unverified**
5. **不得补造历史证据**

### 4.3 证据状态判定
**actual_execution** 必须满足：
- codex-runs 目录存在
- 7 个文件齐全
- command.txt 包含真实命令
- stdout.txt 包含真实输出
- exit_code 与实际一致

**execution_unverified** 适用于：
- 缺少 codex-runs 目录
- 文件不齐全
- 命令或输出是事后编造
- 无法证明真实执行

---

## 5. 与 codex-reports 的关系

### 5.1 codex-runs（原始证据）
- **性质**：Codex CLI 原始输出
- **内容**：命令、输出、错误、退出代码、元数据
- **用途**：证明 Codex 真实执行
- **不可修改**：保存后不得修改

### 5.2 codex-reports（整理报告）
- **性质**：CEO/人工整理的报告
- **内容**：分析、结论、建议、验证方法
- **用途**：提供人类可读的审查结果
- **可修改**：可根据需要更新

### 5.3 关系规则
1. **codex-runs 是原始证据源**
2. **codex-reports 是整理报告**
3. **没有 codex-runs 证据目录，不得声称 Codex 已真实执行**
4. **如果是人工整理，必须明确标注为 Codex-style report**
5. **CEO 读取报告后写入 memory 时，必须标注 actual_execution 或 execution_unverified**

---

## 6. 示例：CODEX-20260514-001 证据目录

### 6.1 目录结构
```
qianchen-os-docs/codex-runs/CODEX-20260514-001/
├── task.yaml
├── command.txt
├── stdout.txt
├── stderr.txt
├── exit-code.txt
├── last-message.md
└── run-metadata.json
```

### 6.2 run-metadata.json 示例
```json
{
  "task_id": "CODEX-20260514-001",
  "task_type": "read-only",
  "repo": "/home/administrator/.openclaw/workspace/ai-company",
  "executor": "Human-Tool",
  "started_at": "2026-05-14T08:25:00Z",
  "finished_at": "2026-05-14T08:30:00Z",
  "codex_cli_version": "1.0.0",
  "sandbox": "read-only",
  "approval_mode": "never",
  "exit_code": 0,
  "usage_limit_detected": false,
  "boundary_violation_detected": false,
  "report_path": "qianchen-os-docs/codex-reports/2026-05-14-read-only-generate-proposals.md",
  "evidence_status": "execution_unverified"
}
```

**注意**：当前 CODEX-20260514-001 标记为 `execution_unverified`，因为缺少真实运行证据。

---

## 7. 版本历史

- **v0.1** (2026-05-14)：初始模板，定义证据目录结构和验证规则

---

**模板状态**：✅ 设计完成  
**下一步**：等待 CEO 审批  
**维护者**：千尘（CEO）