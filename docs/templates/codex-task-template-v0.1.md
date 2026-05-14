# Codex 任务单模板 v0.1

**版本**：0.1  
**设计时间**：2026-05-14 07:50 GMT+8  
**设计者**：千尘（CEO）  
**用途**：CEO 每次调用 Codex 前必须先创建规范任务单  
**状态**：设计阶段，不写代码

---

## 1. 模板目的

让 CEO 每次调用 Codex 前必须先创建规范任务单，避免无边界调用。

---

## 2. YAML 模板

```yaml
# Codex 任务单模板 v0.1
# CEO 每次调用 Codex 前必须填写此模板

# 任务标识
task_id: "CODEX-[YYYYMMDD]-[序号]"  # 示例：CODEX-20260514-001
task_type: "read-only|patch-only|local-verify|PR-review"  # 必填，四选一

# 仓库信息
repo: "仓库路径"  # 示例：/home/administrator/.openclaw/workspace/ai-company

# 目标文件
target_files:  # 必填，数组格式
  - "文件路径1"
  - "文件路径2"
  # 示例：["src/generate_proposals.py", "src/utils.py"]

# 任务范围
scope: "审查/修复范围描述"  # 必填，文本描述
# 示例：审查 generate_proposals 函数是否符合规格说明书要求

# 禁止操作
forbidden:  # 必填，明确禁止的操作
  - "git push"
  - "git commit --amend"
  - "rm -rf"
  - "curl without timeout"
  - "修改 openclaw.json"
  - "修改 *.token/*.env"
  - "修改 ~/.ssh/*"
  - "修改 ~/.gnupg/*"

# 验收标准
acceptance_criteria:  # 必填，数组格式
  - "标准1"
  - "标准2"
  # 示例：["报告包含问题分析", "补丁能正常应用", "验证命令可执行"]

# 执行限制
timeout: 300  # 超时时间（秒），必填，建议 300-600 秒
cost_limit: 0.50  # 成本限制（美元），必填，建议 0.10-1.00 美元
sandbox: "read-only"  # 必填：read-only（只读审查）或 workspace-write（生成补丁）
approval_mode: "never"  # 必填：auto（自动审批）或 manual（手动审批）或 never（从不审批）

# 输出路径
output_path: "输出文件路径"  # 可选，如生成补丁文件
# 示例：qianchen-os-docs/patches/2026-05-14-fix-generate-proposals.patch

# 报告路径
report_path: "报告文件路径"  # 必填，必须在 qianchen-os-docs/codex-reports/
# 示例：qianchen-os-docs/codex-reports/2026-05-14-read-only-generate-proposals.md

# 交付协议
delivery_protocol: "CEO重要消息交付协议v1"  # 必填，固定值
# 流程：报告文件 → CEO读取 → memory沉淀 → Telegram摘要

# 降级策略
fallback:  # 必填
  usage_limit: "GPT"  # usage limit 时转 GPT
  timeout: "Human-Tool"  # 超时转 Human-Tool
  boundary_violation: "GPT"  # 越界尝试转 GPT

# 附加信息（可选）
priority: "P0|P1|P2"  # 优先级
related_task: "关联任务ID"  # 关联任务
notes: "备注信息"  # 备注
```

---

## 3. 任务类型填写说明

### 3.1 read-only（只读审查）
**用途**：分析代码、识别问题、提供审查意见

**填写要点**：
- `task_type`：`read-only`
- `sandbox`：必须为 `read-only`
- `approval_mode`：建议 `never`
- `scope`：明确审查重点和范围
- `target_files`：指定要审查的文件
- `acceptance_criteria`：要求报告包含问题分析、风险识别、改进建议
- `output_path`：可选，通常不需要
- `report_path`：必须，审查报告路径

**示例场景**：
- 审查代码质量
- 识别安全漏洞
- 分析性能问题
- 评估架构合理性

### 3.2 patch-only（生成补丁草稿）
**用途**：生成修复补丁、函数替换代码

**填写要点**：
- `task_type`：`patch-only`
- `sandbox`：必须为 `workspace-write`
- `approval_mode`：建议 `never`（只生成补丁，不直接应用）
- `scope`：明确要修复的问题
- `target_files`：指定要修改的文件
- `acceptance_criteria`：要求补丁能正常应用、包含验证命令
- `output_path`：必须，补丁文件路径
- `report_path`：必须，修复报告路径

**示例场景**：
- 修复 TypeScript 编译错误
- 修复安全漏洞
- 优化性能问题
- 重构代码结构

### 3.3 local-verify（本地验证/测试建议）
**用途**：提供验证命令、测试建议

**填写要点**：
- `task_type`：`local-verify`
- `sandbox`：建议 `read-only`
- `approval_mode`：建议 `never`
- `scope`：明确验证目标和范围
- `target_files`：指定相关文件
- `acceptance_criteria`：要求提供可执行的验证命令、测试用例
- `output_path`：可选，测试脚本路径
- `report_path`：必须，验证报告路径

**示例场景**：
- 提供单元测试建议
- 生成验证命令
- 设计测试策略
- 评估测试覆盖率

### 3.4 PR-review（审查 PR 或 diff）
**用途**：审查代码变更、评估影响

**填写要点**：
- `task_type`：`PR-review`
- `sandbox`：建议 `read-only`
- `approval_mode`：建议 `never`
- `scope`：明确审查重点和变更范围
- `target_files`：指定变更文件
- `acceptance_criteria`：要求分析变更影响、评估风险、提供合并建议
- `output_path`：可选，审查补丁路径
- `report_path`：必须，审查报告路径

**示例场景**：
- 审查 Pull Request
- 评估代码变更
- 分析影响范围
- 提供合并建议

---

## 4. 示例：read-only 审查 generate_proposals

```yaml
# Codex 任务单 - 示例
# 任务：审查 generate_proposals 是否符合规格

# 任务标识
task_id: "CODEX-20260514-001"
task_type: "read-only"

# 仓库信息
repo: "/home/administrator/.openclaw/workspace/ai-company"

# 目标文件
target_files:
  - "coding_agent_server.py"
  - "qianchen-os-docs/docs/specs/generate-proposals-v1.md"

# 任务范围
scope: "只读审查 ai-company/coding_agent_server.py 中的 generate_proposals 函数是否符合 qianchen-os-docs/docs/specs/generate-proposals-v1.md 规格要求，包括输出结构、steps、risks、acceptance_criteria、owner、priority、evidence 字段。"

# 禁止操作
forbidden:
  - "git push"
  - "git commit --amend"
  - "rm -rf"
  - "curl without timeout"
  - "修改 openclaw.json"
  - "修改 *.token/*.env"
  - "修改 ~/.ssh/*"
  - "修改 ~/.gnupg/*"

# 验收标准
acceptance_criteria:
  - "报告包含函数与规格的对比分析"
  - "报告识别不符合规格的地方"
  - "报告提供改进建议"
  - "报告包含验证方法"

# 执行限制
timeout: 300
cost_limit: 0.30
sandbox: "read-only"
approval_mode: "never"

# 输出路径
output_path: ""

# 报告路径
report_path: "qianchen-os-docs/codex-reports/2026-05-14-read-only-generate-proposals.md"

# 交付协议
delivery_protocol: "CEO重要消息交付协议v1"

# 降级策略
fallback:
  usage_limit: "GPT"
  timeout: "Human-Tool"
  boundary_violation: "GPT"

# 附加信息
priority: "P1"
related_task: ""
notes: "重点检查输入参数验证和错误处理机制"
```

---

## 5. 关键规则

### 5.1 报告路径规则
- **必须**：`report_path` 必须在 `qianchen-os-docs/codex-reports/` 目录下
- **格式**：`qianchen-os-docs/codex-reports/YYYY-MM-DD-[task_type]-[来源].md`
- **示例**：`qianchen-os-docs/codex-reports/2026-05-14-read-only-generate-proposals.md`

### 5.2 运行证据规则
- **必须**：创建证据目录 `qianchen-os-docs/codex-runs/<task_id>/`
- **必须**：包含以下文件：
  - `task.yaml`：任务单文件
  - `command.txt`：原始调用命令
  - `stdout.txt`：标准输出
  - `stderr.txt`：标准错误
  - `exit-code.txt`：退出代码
  - `last-message.md`：最后消息
  - `run-metadata.json`：运行元数据
- **必须**：run-metadata.json 包含 task_id、started_at、finished_at、executor、codex_cli_version、sandbox、approval_mode、exit_code、usage_limit_detected、report_path
- **铁律**：没有 Codex 原始调用证据不算执行完成
- **铁律**：没有运行证据不得声称已执行
- **铁律**：必须区分 actual_execution 和 execution_unverified

### 5.3 安全规则
- **不允许**：直接改 main
- **不允许**：自动 git push
- **不允许**：碰 token/.env/openclaw.json
- **不允许**：碰 ~/.ssh/*、~/.gnupg/*
- **必须**：所有 curl 命令有 timeout
- **必须**：所有命令有超时限制

### 5.3 交付协议
- **必须**：走 CEO 重要消息交付协议 v1
- **流程**：报告文件 → CEO 读取 → memory 沉淀 → Telegram 摘要
- **禁止**：只发 Telegram 或 message_bus

### 5.4 降级策略
- **usage limit**：转 GPT
- **timeout**：转 Human-Tool
- **boundary_violation**：转 GPT
- **原则**：按任务类型路由，避免无边界调用

---

## 6. 版本历史

- **v0.1** (2026-05-14)：初始模板，定义基本字段和填写说明

---

**模板状态**：✅ 设计完成  
**下一步**：等待 CEO 审批  
**维护者**：千尘（CEO）