# 千尘 OS 文档库

> 公共技术资料沉淀，非敏感内容

## 目录结构

```
qianchen-os-docs/
├── docs/
│   ├── architecture/          # 架构设计
│   ├── reports/               # 技术报告
│   ├── decisions/             # 决策记录 (ADR)
│   └── roadmap/               # 路线图
├── task-logs/
│   └── public/                # 公开任务日志
├── codex-reports/             # Codex 执行报告
└── README.md
```

## 使用规范

### 上传内容
- 架构设计文档
- 技术报告
- 决策记录 (ADR)
- 任务日志（脱敏）
- Codex 执行报告
- 路线图

### 禁止内容
- API Token / Key
- 密码 / Secret
- 用户隐私聊天原文
- 私有配置文件
- 真实个人敏感数据
- 完整 openclaw.json

### 敏感信息处理
- API 端点：用 `API_ENDPOINT_*` 占位符
- Token：用 `REDACTED` 或删除
- 用户 ID：用 `USER_ID_*` 占位符
- 密钥：完全删除

## 汇报规范

ceo 汇报时：
1. 只给摘要（≤5句话）
2. 附仓库路径链接
3. 不贴全文

## 维护者

- 千尘 🌌 (ceo)
- 雨陌 (投资人)

---

*创建时间：2026-05-13*
