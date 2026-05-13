# 报告文件路径规范 v1

**版本**：1.0  
**生效日期**：2026-05-14  
**制定者**：千尘（CEO）  
**状态**：正式生效

---

## 1. 规范目的

确保 Codex 产出文件有统一的命名和存储规范，便于查找、管理和归档。

---

## 2. 目录结构

```
qianchen-os-docs/
├── reports/                    # 审查报告、分析报告
│   ├── YYYY-MM-DD-[类型]-[来源].md
│   └── YYYY-MM-DD-[类型]-[来源].json
├── patches/                    # 补丁文件
│   └── YYYY-MM-DD-[问题]-[文件].patch
├── suggestions/                # 测试建议、架构建议
│   └── YYYY-MM-DD-[功能]-[策略].md
└── docs/
    ├── architecture/           # 架构设计文档
    ├── standards/              # 规范文档
    ├── guides/                 # 指南文档
    └── reports/                # 历史报告归档
```

---

## 3. 命名规范

### 3.1 基本格式
```
YYYY-MM-DD-[类型]-[来源].扩展名
```

### 3.2 字段说明

**日期 (YYYY-MM-DD)**
- 格式：年-月-日
- 示例：2026-05-14
- 用途：按时间排序和查找

**类型 (Type)**
- `codex-review`：Codex 代码审查
- `codex-patch`：Codex 补丁生成
- `codex-test`：Codex 测试建议
- `fault-analysis`：故障分析
- `performance`：性能分析
- `security`：安全审查
- `architecture`：架构设计
- `implementation`：实现方案

**来源 (Source)**
- `codex`：Codex 生成
- `ceo`：CEO 创建
- `qa`：qa_agent 验收
- `tools`：tools_agent 执行
- `memory`：memory_agent 沉淀
- `manual`：手动创建

**扩展名**
- `.md`：Markdown 文档（主要）
- `.json`：结构化数据
- `.patch`：补丁文件
- `.log`：日志文件

### 3.3 示例

**Codex 审查报告**：
```
2026-05-14-codex-review-codex.md
```

**Codex 补丁文件**：
```
2026-05-14-typescript-fix-codex.patch
```

**故障分析报告**：
```
2026-05-14-fault-analysis-ceo.md
```

**测试建议**：
```
2026-05-14-unit-test-codex.md
```

---

## 4. 内容格式

### 4.1 Markdown 报告格式
```markdown
# [报告标题]

**报告时间**：YYYY-MM-DD HH:MM GMT+8  
**报告人**：[来源]  
**报告对象**：[接收者]  
**任务ID**：T[ID]（如适用）

---

## 1. 摘要
[简要描述报告内容]

## 2. 详细内容
[详细分析、发现、建议]

## 3. 证据
[日志、截图、测试结果]

## 4. 结论
[总结和下一步建议]

## 5. 附件
[相关文件链接]
```

### 4.2 JSON 报告格式
```json
{
  "timestamp": "ISO-8601",
  "type": "report-type",
  "source": "source-name",
  "task_id": "T[ID]",
  "summary": "简要描述",
  "details": {},
  "evidence": [],
  "conclusions": [],
  "attachments": []
}
```

### 4.3 补丁文件格式
```diff
--- a/path/to/file.js
+++ b/path/to/file.js
@@ -123,7 +123,7 @@
  existing code
- old line
+ new line
  more existing code
```

---

## 5. 存储规则

### 5.1 保留期限
- **最近 7 天**：所有文件保留
- **7-30 天**：重要文件保留，普通文件可归档
- **30 天以上**：归档到 `docs/reports/` 或删除

### 5.2 归档规则
1. 每月 1 日归档上月文件
2. 归档路径：`docs/reports/YYYY-MM/`
3. 保持原始文件名
4. 创建归档索引文件

### 5.3 清理规则
1. 临时文件（.tmp）立即删除
2. 重复文件保留最新版本
3. 空文件删除
4. 损坏文件标记并隔离

---

## 6. 访问权限

### 6.1 读取权限
- **CEO**：所有文件
- **子Agent**：相关任务文件
- **用户**：通过 CEO 授权

### 6.2 写入权限
- **Codex**：`reports/`、`patches/`、`suggestions/`
- **子Agent**：相关任务目录
- **CEO**：所有目录

### 6.3 删除权限
- **CEO**：所有文件
- **qa_agent**：验收不合格文件
- **自动清理**：过期文件

---

## 7. 版本控制

### 7.1 文件版本
- 使用 Git 管理重要文件
- 提交信息格式：`[类型] [来源] [简要描述]`
- 示例：`[codex-review] [codex] TypeScript 编译错误分析`

### 7.2 版本标签
- 重要版本打标签：`v1.0`、`v1.1`
- 标签格式：`规范名-版本号`
- 示例：`report-path-standard-v1.0`

---

## 8. 验收标准

### 8.1 命名验收
- [ ] 符合 YYYY-MM-DD-[类型]-[来源].扩展名 格式
- [ ] 日期是实际创建日期
- [ ] 类型和来源正确标识
- [ ] 扩展名与内容匹配

### 8.2 内容验收
- [ ] 有完整的报告头信息
- [ ] 有摘要和详细内容
- [ ] 有证据和结论
- [ ] 格式清晰易读

### 8.3 存储验收
- [ ] 存储在正确目录
- [ ] 文件名唯一
- [ ] 权限设置正确
- [ ] 可正常访问

---

## 9. 异常处理

### 9.1 命名冲突
- 添加序号：`-1`、`-2`
- 示例：`2026-05-14-codex-review-codex-1.md`

### 9.2 内容缺失
- 标记为 `[待补充]`
- 创建后续任务补充
- 记录缺失原因

### 9.3 文件损坏
- 标记为 `[损坏]`
- 尝试修复或重建
- 记录损坏原因

---

## 10. 版本历史

- **v1.0** (2026-05-14)：初始版本，建立基本命名和存储规范

---

**规范状态**：✅ 正式生效  
**下次审查**：2026-05-21  
**维护者**：千尘（CEO）