# V5 行为学习输入链路修复方案 v0.1

**版本**：0.1  
**设计时间**：2026-05-15 02:15 GMT+8  
**设计者**：千尘（CEO）  
**状态**：设计阶段，不改代码

---

## 1. 问题背景

### 1.1 当前事故
- **cron:c7b8b6b1-3249-40db-86c7-8c4a987cbb93** 运行成功
- **输出**："未找到对话对，今日无新行为模型"
- **实际情况**：今日（2026-05-14）有大量可学习行为反馈
- **结论**：脚本运行成功，但行为学习链路失效

### 1.2 根本原因
1. **日期过滤错误**：脚本使用 `datetime.now()` 获取今天的日期（2026-05-15），但对话发生在昨天（2026-05-14）
2. **输入源不完整**：只读取 `memory/YYYY-MM-DD.md`，忽略了其他输入源
3. **解析规则不匹配**：期望以"千尘"、"雨陌"开头的对话格式，但记忆文件是工作日志格式

---

## 2. 输入源分层

### 2.1 输入源优先级
| 优先级 | 输入源 | 用途 | 示例 |
|--------|--------|------|------|
| 1 | 对话原文/会话日志 | 优先学习证据 | OpenClaw 会话记录 |
| 2 | memory/YYYY-MM-DD.md | 学习线索 | 工作日志、决策记录 |
| 3 | message_bus.json | 事件与状态副本 | 派工、ACK、状态变更 |
| 4 | qianchen-os-docs/reports | 事实报告 | Codex 审查报告、故障分析 |

### 2.2 输入源说明

**对话原文/会话日志（优先级最高）**
- **来源**：OpenClaw 会话记录、Telegram 消息历史
- **格式**：原始对话，包含 user/assistant 消息
- **优点**：最直接的学习证据
- **缺点**：可能需要 API 调用或文件解析

**memory/YYYY-MM-DD.md（学习线索）**
- **来源**：CEO 每日工作日志
- **格式**：工作日志格式（`**时间**：`、`**执行者**：`、`**决策**：`）
- **优点**：结构化，包含决策和修正
- **缺点**：不是原始对话，需要解析

**message_bus.json（事件与状态副本）**
- **来源**：AI 公司中台消息总线
- **格式**：JSON 消息记录
- **优点**：包含派工、ACK、状态变更
- **缺点**：不包含对话内容

**qianchen-os-docs/reports（事实报告）**
- **来源**：Codex 审查报告、故障分析、设计文档
- **格式**：Markdown 报告
- **优点**：包含详细分析和结论
- **缺点**：不是对话，是分析结果

---

## 3. 日期策略

### 3.1 当前问题
- **只读 datetime.now()**：脚本运行在 2026-05-15 02:10，但对话发生在 2026-05-14
- **时区问题**：未考虑 Asia/Singapore 时区
- **无回溯能力**：无法查找最近 N 天的文件

### 3.2 修复方案

**支持最近 N 天**
```python
# 默认查找最近 3 天的文件
LOOKBACK_DAYS = 3

def get_memory_files(lookback_days=LOOKBACK_DAYS):
    """获取最近 N 天的记忆文件"""
    files = []
    for i in range(lookback_days):
        date = datetime.now() - timedelta(days=i)
        file_path = MEMORY_DIR / f"{date.strftime('%Y-%m-%d')}.md"
        if file_path.exists():
            files.append(file_path)
    return files
```

**支持 Asia/Singapore 时区**
```python
from zoneinfo import ZoneInfo

def get_today_singapore():
    """获取新加坡时区的今天日期"""
    return datetime.now(ZoneInfo("Asia/Singapore")).strftime("%Y-%m-%d")
```

**支持手动指定日期**
```python
# 命令行参数支持
# python3 behavior_learner.py --date 2026-05-14
# python3 behavior_learner.py --days 7
```

---

## 4. 解析规则

### 4.1 当前问题
- **只依赖行首标记**：期望以"千尘"、"雨陌"开头的行
- **不兼容工作日志格式**：记忆文件使用 `**时间**：`、`**执行者**：` 等标记
- **不兼容报告格式**：报告文件使用 `## 标题`、`**结论**：` 等标记

### 4.2 修复方案

**兼容工作日志格式**
```python
# 匹配工作日志中的决策和修正
WORKLOG_PATTERNS = [
    # 决策模式
    r'\*\*决策\*\*[：:]\s*(.+)',
    r'\*\*结论\*\*[：:]\s*(.+)',
    r'\*\*判断\*\*[：:]\s*(.+)',
    
    # 问题模式
    r'\*\*问题\*\*[：:]\s*(.+)',
    r'\*\*原因\*\*[：:]\s*(.+)',
    r'\*\*根因\*\*[：:]\s*(.+)',
    
    # 修正模式
    r'\*\*修正\*\*[：:]\s*(.+)',
    r'\*\*改为\*\*[：:]\s*(.+)',
    r'\*\*从\*\*[：:].+改为[：:]\s*(.+)',
    
    # 铁律模式
    r'\*\*铁律\*\*[：:]\s*(.+)',
    r'没有.+不得.+',
]
```

**兼容报告格式**
```python
# 匹配报告中的关键部分
REPORT_PATTERNS = [
    # 审查结论
    r'## 审查结论\s*\n(.+?)(?=\n##|\Z)',
    # 关键发现
    r'## 关键发现\s*\n(.+?)(?=\n##|\Z)',
    # 改进建议
    r'## 改进建议\s*\n(.+?)(?=\n##|\Z)',
]
```

**兼容"判断/问题/修正/铁律/下一步"结构**
```python
# 匹配结构化内容
STRUCTURED_PATTERNS = {
    "judgment": r'【判断】\s*\n(.+?)(?=\n【|\Z)',
    "problem": r'【问题】\s*\n(.+?)(?=\n【|\Z)',
    "correction": r'【修正】\s*\n(.+?)(?=\n【|\Z)',
    "iron_rule": r'【铁律】\s*\n(.+?)(?=\n【|\Z)',
    "next_step": r'【下一步】\s*\n(.+?)(?=\n【|\Z)',
}
```

---

## 5. 学习样本判定

### 5.1 当前问题
- **只依赖对话格式**：期望千尘回复 → 雨陌反馈的对话对
- **忽略其他学习信号**：问题指出、错误承认、状态修正等

### 5.2 修复方案

**学习样本类型**
| 类型 | 说明 | 示例 |
|------|------|------|
| 用户指出问题 | 雨陌指出 OpenClaw 的错误或不足 | "这次验收暂停"、"这个方案有问题" |
| OpenClaw 承认错误 | CEO 承认错误或接受反馈 | "承认事实"、"状态修正" |
| 状态被修正 | 任务状态、工作状态被修改 | "改为：learning_parser_failed" |
| 新铁律产生 | 新的规则或约束被确立 | "新增铁律：没有 ACK 不算派工" |
| 后续行为发生改变 | OpenClaw 的行为因反馈而改变 | "已修正"、"已更新" |

**学习样本判定规则**
```python
def is_learning_sample(content: str) -> Tuple[bool, str, float]:
    """
    判断是否为学习样本
    返回: (是否样本, 样本类型, 置信度)
    """
    # 用户指出问题
    if re.search(r'(问题|错误|不对|暂停|修正)', content):
        return (True, "user_pointed_problem", 0.8)
    
    # OpenClaw 承认错误
    if re.search(r'(承认|接受|修正|改为|状态修正)', content):
        return (True, "openclaw_admitted_error", 0.7)
    
    # 新铁律产生
    if re.search(r'(铁律|不得|必须|禁止|新增铁律)', content):
        return (True, "new_iron_rule", 0.9)
    
    # 状态被修正
    if re.search(r'(从.+改为|状态改为|修正为)', content):
        return (True, "status_corrected", 0.8)
    
    return (False, "no_sample", 0.0)
```

---

## 6. 运行报告必须包含

### 6.1 当前问题
- **只输出"未找到对话对"**：无详细统计
- **无输入统计**：不知道读取了哪些文件
- **无解析统计**：不知道解析了多少消息
- **无过滤原因**：不知道为什么没有学习样本

### 6.2 修复方案

**运行报告格式**
```json
{
  "run_id": "behavior_learn_20260515_021500",
  "timestamp": "2026-05-15T02:15:00+08:00",
  "status": "learning_completed",
  
  "input_summary": {
    "files_read": [
      {"path": "memory/2026-05-14.md", "size": 10195, "exists": true},
      {"path": "memory/2026-05-15.md", "size": 852, "exists": true}
    ],
    "total_files": 2,
    "total_chars": 11047
  },
  
  "parsing_summary": {
    "total_lines": 500,
    "parsed_messages": 45,
    "user_messages": 20,
    "assistant_messages": 25,
    "conversation_pairs": 15
  },
  
  "filter_summary": {
    "total_candidates": 15,
    "filtered_out": 12,
    "filter_reasons": {
      "low_confidence": 5,
      "neutral_feedback": 4,
      "duplicate": 3
    },
    "final_samples": 3
  },
  
  "output_summary": {
    "models_created": 1,
    "models_updated": 2,
    "models_reinforced": 1,
    "models_degraded": 0
  },
  
  "diagnostics": {
    "timezone": "Asia/Singapore",
    "lookback_days": 3,
    "date_range": ["2026-05-13", "2026-05-14", "2026-05-15"],
    "parsing_rules_used": ["worklog", "report", "structured"]
  }
}
```

---

## 7. 状态枚举

### 7.1 学习状态定义
| 状态 | 说明 | 触发条件 |
|------|------|----------|
| `learning_completed` | 学习完成，有输出 | 找到学习样本，创建/更新了模型 |
| `learning_no_candidate` | 学习完成，无候选 | 读取了输入，但无有效学习样本 |
| `learning_unverified` | 学习未验证 | 脚本运行成功，但学习结果不可信 |
| `learning_input_missing` | 输入缺失 | 未找到输入文件或文件为空 |
| `learning_parser_failed` | 解析失败 | 解析规则不匹配，无法提取样本 |

### 7.2 状态转换规则
```
输入文件存在？
├─ 否 → learning_input_missing
└─ 是 → 解析成功？
    ├─ 否 → learning_parser_failed
    └─ 是 → 找到学习样本？
        ├─ 否 → learning_no_candidate
        └─ 是 → 学习结果有效？
            ├─ 否 → learning_unverified
            └─ 是 → learning_completed
```

---

## 8. 新增铁律

### 8.1 学习链路铁律
1. **没有输入统计、解析统计、过滤原因，不得声称行为学习正常完成**
2. **没有读到有效对话样本，不得声称"今日无新行为模型"**
3. **脚本运行成功 ≠ 学习完成**
4. **必须区分 learning_completed 和 learning_no_candidate**

### 8.2 输入源铁律
1. **不得只依赖 datetime.now() 获取日期**
2. **不得只读取一个记忆文件**
3. **不得只依赖对话格式解析**
4. **必须支持最近 N 天回溯**

### 8.3 输出报告铁律
1. **必须包含输入文件列表**
2. **必须包含解析统计**
3. **必须包含过滤原因**
4. **必须包含状态枚举**

---

## 9. 版本历史

- **v0.1** (2026-05-15)：初始设计，定义输入链路修复方案

---

**设计状态**：✅ 完成  
**下一步**：等待 CEO 审批  
**维护者**：千尘（CEO）