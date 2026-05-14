# V5 行为学习输入链路实现规格 v0.1

**版本**：0.1  
**设计时间**：2026-05-15 02:55 GMT+8  
**设计者**：千尘（CEO）  
**状态**：实现规格，不直接改代码

---

## 1. 需要修改的函数列表

### 1.1 现有函数修改

| 函数名 | 当前问题 | 修改内容 |
|--------|----------|----------|
| `_get_today_memory_file` | 只读 datetime.now() | 支持 --date、--days、--timezone 参数 |
| `_extract_conversation_pairs` | 只匹配"千尘：/雨陌："行首 | 调用多个解析器模块 |
| `learn_from_today` | 无统计输出 | 输出完整运行报告 |
| `__main__` | 无参数支持 | 添加 argparse 参数解析 |

### 1.2 新增函数

| 函数名 | 用途 |
|--------|------|
| `get_memory_files_by_params` | 根据参数获取输入文件列表 |
| `parse_with_all_parsers` | 调用所有解析器并合并结果 |
| `generate_run_report` | 生成运行报告 JSON |
| `run_dry_run` | dry-run 模式，不写入模型 |

---

## 2. 新增命令行参数

### 2.1 参数定义

```python
import argparse

parser = argparse.ArgumentParser(description="V5 行为学习器")
parser.add_argument(
    "--date", 
    type=str, 
    help="指定日期，格式：YYYY-MM-DD（例如：2026-05-14）"
)
parser.add_argument(
    "--days", 
    type=int, 
    default=3,
    help="回溯天数，默认 3 天"
)
parser.add_argument(
    "--timezone", 
    type=str, 
    default="Asia/Singapore",
    help="时区，默认 Asia/Singapore"
)
parser.add_argument(
    "--dry-run", 
    action="store_true",
    help="只输出报告，不写入模型"
)
parser.add_argument(
    "--output-report", 
    type=str,
    help="输出报告路径（可选）"
)
```

### 2.2 参数使用示例

```bash
# 默认：查找最近 3 天（Asia/Singapore 时区）
python3 behavior_learner.py

# 指定日期
python3 behavior_learner.py --date 2026-05-14

# 回溯 7 天
python3 behavior_learner.py --days 7

# 指定时区
python3 behavior_learner.py --timezone Asia/Shanghai

# dry-run 模式（不写入模型）
python3 behavior_learner.py --dry-run

# 输出报告到文件
python3 behavior_learner.py --dry-run --output-report /tmp/report.json
```

---

## 3. 输入文件发现逻辑

### 3.1 函数签名

```python
def get_memory_files_by_params(
    date: Optional[str] = None,
    days: int = 3,
    timezone: str = "Asia/Singapore"
) -> List[Path]:
    """
    根据参数获取输入文件列表
    
    参数:
        date: 指定日期（YYYY-MM-DD），优先级最高
        days: 回溯天数，当 date 为 None 时使用
        timezone: 时区
    
    返回:
        文件路径列表，按日期倒序（最新在前）
    """
```

### 3.2 发现逻辑

```
输入参数
├─ date 指定？
│  ├─ 是 → 只读取 memory/{date}.md
│  └─ 否 → 继续
├─ 获取当前时区的日期
├─ 回溯 days 天
├─ 检查每个日期的 memory/{date}.md
├─ 文件存在？
│  ├─ 是 → 添加到列表
│  └─ 否 → 跳过
└─ 返回文件列表（按日期倒序）
```

### 3.3 代码示例

```python
from datetime import datetime, timedelta
from zoneinfo import ZoneInfo

def get_memory_files_by_params(
    date: Optional[str] = None,
    days: int = 3,
    timezone: str = "Asia/Singapore"
) -> List[Path]:
    files = []
    
    if date:
        # 指定日期模式
        file_path = MEMORY_DIR / f"{date}.md"
        if file_path.exists():
            files.append(file_path)
    else:
        # 回溯模式
        tz = ZoneInfo(timezone)
        today = datetime.now(tz).date()
        
        for i in range(days):
            check_date = today - timedelta(days=i)
            file_path = MEMORY_DIR / f"{check_date.strftime('%Y-%m-%d')}.md"
            if file_path.exists():
                files.append(file_path)
    
    return files
```

---

## 4. 解析器模块划分

### 4.1 模块结构

```
memory_v4/
├── behavior_learner.py          # 主脚本
└── parsers/                     # 解析器模块（新增）
    ├── __init__.py
    ├── base_parser.py           # 基类
    ├── worklog_parser.py        # 工作日志解析器
    ├── report_parser.py         # 报告解析器
    ├── structured_parser.py     # 结构化内容解析器
    └── conversation_parser.py   # 对话解析器
```

### 4.2 解析器基类

```python
# parsers/base_parser.py
from abc import ABC, abstractmethod
from typing import List, Tuple, Dict

class BaseParser(ABC):
    """解析器基类"""
    
    @property
    @abstractmethod
    def name(self) -> str:
        """解析器名称"""
        pass
    
    @abstractmethod
    def parse(self, content: str) -> List[Dict]:
        """
        解析内容，返回候选样本列表
        
        返回:
            [{
                "type": "sample_type",
                "content": "sample_content",
                "confidence": 0.8,
                "source": "parser_name",
                "context": "surrounding_context"
            }]
        """
        pass
    
    @abstractmethod
    def get_stats(self) -> Dict:
        """获取解析统计"""
        pass
```

### 4.3 工作日志解析器

```python
# parsers/worklog_parser.py
import re
from typing import List, Dict

class WorklogParser(BaseParser):
    """解析工作日志格式"""
    
    @property
    def name(self) -> str:
        return "worklog_parser"
    
    # 匹配模式
    PATTERNS = {
        "judgment": [
            r'\*\*判断\*\*[：:]\s*(.+)',
            r'\*\*结论\*\*[：:]\s*(.+)',
            r'\*\*决策\*\*[：:]\s*(.+)',
        ],
        "problem": [
            r'\*\*问题\*\*[：:]\s*(.+)',
            r'\*\*原因\*\*[：:]\s*(.+)',
            r'\*\*根因\*\*[：:]\s*(.+)',
        ],
        "correction": [
            r'\*\*修正\*\*[：:]\s*(.+)',
            r'\*\*改为\*\*[：:]\s*(.+)',
            r'从\s*(.+?)\s*改为\s*(.+)',
        ],
        "iron_rule": [
            r'\*\*铁律\*\*[：:]\s*(.+)',
            r'(没有.+不得.+)',
            r'(必须.+)',
            r'(禁止.+)',
        ],
    }
    
    def parse(self, content: str) -> List[Dict]:
        samples = []
        lines = content.split('\n')
        
        for i, line in enumerate(lines):
            for sample_type, patterns in self.PATTERNS.items():
                for pattern in patterns:
                    match = re.search(pattern, line)
                    if match:
                        samples.append({
                            "type": sample_type,
                            "content": match.group(1).strip(),
                            "confidence": 0.8,
                            "source": self.name,
                            "context": '\n'.join(lines[max(0, i-2):min(len(lines), i+3)])
                        })
        
        return samples
    
    def get_stats(self) -> Dict:
        return {"patterns_matched": self._stats}
```

### 4.4 报告解析器

```python
# parsers/report_parser.py
class ReportParser(BaseParser):
    """解析报告格式（## 标题、**结论**：等）"""
    
    @property
    def name(self) -> str:
        return "report_parser"
    
    PATTERNS = {
        "conclusion": [
            r'## 审查结论\s*\n(.+?)(?=\n##|\Z)',
            r'## 结论\s*\n(.+?)(?=\n##|\Z)',
        ],
        "finding": [
            r'## 关键发现\s*\n(.+?)(?=\n##|\Z)',
            r'## 发现\s*\n(.+?)(?=\n##|\Z)',
        ],
        "recommendation": [
            r'## 改进建议\s*\n(.+?)(?=\n##|\Z)',
            r'## 建议\s*\n(.+?)(?=\n##|\Z)',
        ],
    }
    
    def parse(self, content: str) -> List[Dict]:
        # 实现类似 worklog_parser
        pass
```

### 4.5 结构化内容解析器

```python
# parsers/structured_parser.py
class StructuredParser(BaseParser):
    """解析【判断】【问题】【修正】等结构化内容"""
    
    @property
    def name(self) -> str:
        return "structured_parser"
    
    PATTERNS = {
        "judgment": r'【判断】\s*\n(.+?)(?=\n【|\Z)',
        "problem": r'【问题】\s*\n(.+?)(?=\n【|\Z)',
        "correction": r'【修正】\s*\n(.+?)(?=\n【|\Z)',
        "iron_rule": r'【铁律】\s*\n(.+?)(?=\n【|\Z)',
        "next_step": r'【下一步】\s*\n(.+?)(?=\n【|\Z)',
    }
    
    def parse(self, content: str) -> List[Dict]:
        # 实现类似 worklog_parser
        pass
```

### 4.6 对话解析器

```python
# parsers/conversation_parser.py
class ConversationParser(BaseParser):
    """解析原始对话格式（千尘：、雨陌：、user:、assistant:）"""
    
    @property
    def name(self) -> str:
        return "conversation_parser"
    
    PATTERNS = [
        r'^(千尘|我|assistant)[：:]\s*(.+)',
        r'^(雨陌|user)[：:]\s*(.+)',
    ]
    
    def parse(self, content: str) -> List[Dict]:
        # 提取对话对：千尘回复 → 雨陌反馈
        pass
```

---

## 5. 运行报告 JSON 格式

### 5.0 状态枚举定义

| 状态 | 说明 | 触发条件 |
|------|------|----------|
| `learning_completed` | 学习完成，有输出 | 正式运行，找到学习样本，创建/更新了模型 |
| `learning_no_candidate` | 学习完成，无候选 | 正式运行，读取到有效样本，但最终无有效候选 |
| `learning_unverified` | 学习未验证 | 脚本运行成功，但学习结果不可信 |
| `learning_input_missing` | 输入缺失 | 未找到输入文件或文件为空 |
| `learning_parser_failed` | 解析失败 | 读取到文件但 total_samples = 0 |
| `dry_run_completed` | dry-run 完成 | dry-run 模式，读取到文件且 total_samples > 0 |

**关键区分**：
- `dry_run_completed`：dry-run 模式，只验证输入链路和解析链路，不代表学习完成
- `learning_no_candidate`：正式运行，读取到有效样本但最终无有效候选
- `learning_parser_failed`：读取到文件但解析失败（total_samples = 0）

### 5.1 完整格式

```json
{
  "run_id": "behavior_learn_20260515_025500",
  "timestamp": "2026-05-15T02:55:00+08:00",
  "status": "learning_completed",
  
  "input_summary": {
    "files_read": [
      {
        "path": "memory/2026-05-14.md",
        "size": 10195,
        "exists": true,
        "date": "2026-05-14"
      },
      {
        "path": "memory/2026-05-15.md",
        "size": 852,
        "exists": true,
        "date": "2026-05-15"
      }
    ],
    "total_files": 2,
    "total_chars": 11047,
    "date_range": ["2026-05-14", "2026-05-15"],
    "timezone": "Asia/Singapore"
  },
  
  "parsing_summary": {
    "parsers_used": ["worklog_parser", "report_parser", "structured_parser", "conversation_parser"],
    "total_lines": 500,
    "parsed_samples": 45,
    "by_parser": {
      "worklog_parser": 20,
      "report_parser": 10,
      "structured_parser": 10,
      "conversation_parser": 5
    },
    "by_type": {
      "judgment": 15,
      "problem": 10,
      "correction": 8,
      "iron_rule": 7,
      "conclusion": 5
    }
  },
  
  "filter_summary": {
    "total_candidates": 45,
    "filtered_out": 35,
    "filter_reasons": {
      "low_confidence": 15,
      "neutral_feedback": 10,
      "duplicate": 5,
      "no_feedback_signal": 5
    },
    "final_samples": 10
  },
  
  "output_summary": {
    "dry_run": false,
    "models_created": 2,
    "models_updated": 3,
    "models_reinforced": 1,
    "models_degraded": 0,
    "models_json_modified": true,
    "git_committed": true
  },
  
  "diagnostics": {
    "parameters": {
      "date": null,
      "days": 3,
      "timezone": "Asia/Singapore",
      "dry_run": false
    },
    "execution_time_ms": 1500,
    "memory_usage_mb": 50
  }
}
```

### 5.2 最简格式（dry-run）

```json
{
  "run_id": "behavior_learn_20260515_025500_dry",
  "timestamp": "2026-05-15T02:55:00+08:00",
  "status": "dry_run_completed",
  "dry_run": true,
  
  "input_summary": {
    "files_read": [{"path": "memory/2026-05-14.md", "size": 10195}],
    "total_files": 1,
    "total_chars": 10195
  },
  
  "parsing_summary": {
    "total_samples": 45,
    "by_parser": {"worklog_parser": 20, "structured_parser": 25}
  },
  
  "filter_summary": {
    "total_candidates": 45,
    "final_samples": 0,
    "filter_reasons": {"low_confidence": 30, "neutral_feedback": 15}
  },
  
  "output_summary": {
    "models_json_modified": false
  }
}
```

---

## 6. dry-run 验证流程

### 6.1 流程图

```
开始
├─ 解析参数
├─ 获取输入文件列表
├─ 读取每个文件
├─ 调用所有解析器
├─ 合并解析结果
├─ 过滤低质量样本
├─ 生成运行报告
├─ 输出报告到 stdout
├─ --output-report 指定？
│  ├─ 是 → 写入文件
│  └─ 否 → 跳过
└─ 结束（不写入 models.json）
```

### 6.2 关键约束

1. **不写入 models.json**：dry-run 模式下，`model_engine.save_models()` 不被调用
2. **不修改 model-changes.jsonl**：dry-run 模式下，不记录变更日志
3. **不执行 git commit**：dry-run 模式下，不提交代码
4. **只输出报告**：dry-run 模式下，只输出 JSON 报告

### 6.3 代码示例

```python
def run_dry_run(args) -> Dict:
    """dry-run 模式，不写入模型"""
    
    # 1. 获取输入文件
    files = get_memory_files_by_params(
        date=args.date,
        days=args.days,
        timezone=args.timezone
    )
    
    # 2. 解析所有文件
    all_samples = []
    for file_path in files:
        content = file_path.read_text(encoding='utf-8')
        samples = parse_with_all_parsers(content)
        all_samples.extend(samples)
    
    # 3. 过滤样本
    filtered_samples = filter_samples(all_samples)
    
    # 4. 生成报告
    report = generate_run_report(
        files=files,
        all_samples=all_samples,
        filtered_samples=filtered_samples,
        dry_run=True
    )
    
    # 5. 输出报告
    print(json.dumps(report, indent=2, ensure_ascii=False))
    
    # 6. 写入文件（如果指定）
    if args.output_report:
        Path(args.output_report).write_text(
            json.dumps(report, indent=2, ensure_ascii=False)
        )
    
    return report
```

---

## 7. 第一轮验收样本

### 7.1 验收目标

**文件**：`memory/2026-05-14.md`
**大小**：10195 字节
**日期**：2026-05-14

### 7.2 预期输出

运行命令：
```bash
python3 behavior_learner.py --date 2026-05-14 --dry-run
```

预期报告（dry-run 成功）：
```json
{
  "status": "dry_run_completed",
  "dry_run": true,
  "input_summary": {
    "files_read": [{"path": "memory/2026-05-14.md", "size": 10195}],
    "total_files": 1,
    "total_chars": 10195
  },
  "parsing_summary": {
    "total_samples": ">0"
  },
  "filter_summary": {
    "total_candidates": ">0",
    "final_samples": ">=0"
  },
  "output_summary": {
    "models_json_modified": false
  }
}
```

**注意**：dry-run 只验证输入链路和解析链路，不代表学习完成。

### 7.3 关键验证点

1. **能读取文件**：`input_summary.files_read` 包含 `memory/2026-05-14.md`
2. **能解析内容**：`parsing_summary.total_samples > 0`
3. **能输出统计**：报告包含 `input_summary`、`parsing_summary`、`filter_summary`
4. **不再只输出"未找到对话对"**：输出完整 JSON 报告
5. **dry-run 不修改模型**：`models_json_modified: false`

---

## 8. 验收标准

### 8.1 功能验收

| 验收项 | 标准 | 验证方法 |
|--------|------|----------|
| 能读取 2026-05-14.md | `input_summary.files_read` 包含该文件 | 运行 --date 2026-05-14 --dry-run |
| 能输出 input_summary | 报告包含 `input_summary` 字段 | 检查 JSON 输出 |
| 能输出 parsing_summary | 报告包含 `parsing_summary.total_samples > 0` | 检查 JSON 输出 |
| 能输出 filter_summary | 报告包含 `filter_summary` 字段 | 检查 JSON 输出 |
| 不再只输出"未找到对话对" | 输出完整 JSON 报告 | 检查 stdout |
| dry-run 不修改 models.json | `output_summary.models_json_modified: false` | 检查文件修改时间 |

### 8.2 参数验收

| 参数 | 验收标准 |
|------|----------|
| `--date 2026-05-14` | 只读取 memory/2026-05-14.md |
| `--days 7` | 读取最近 7 天的文件 |
| `--timezone Asia/Shanghai` | 使用上海时区计算日期 |
| `--dry-run` | 不写入 models.json |
| `--output-report /tmp/report.json` | 报告写入指定文件 |

### 8.3 解析器验收

| 解析器 | 验收标准 |
|--------|----------|
| worklog_parser | 能解析 `**判断**：`、`**问题**：` 等格式 |
| report_parser | 能解析 `## 审查结论` 等格式 |
| structured_parser | 能解析 `【判断】`、`【问题】` 等格式 |
| conversation_parser | 能解析 `千尘：`、`雨陌：` 等格式 |

---

## 9. 禁止事项

### 9.1 实现阶段禁止

1. ❌ **不直接改 models.json**：dry-run 模式下禁止写入
2. ❌ **不生成假候选**：如果无有效样本，输出 `learning_no_candidate`
3. ❌ **不把脚本运行成功当学习完成**：必须区分 `learning_completed` 和 `learning_no_candidate`
4. ❌ **不跳过统计输出**：必须输出完整的运行报告

### 9.2 验收阶段禁止

1. ❌ **不跳过 dry-run 测试**：必须先通过 dry-run 验证
2. ❌ **不跳过 2026-05-14.md 测试**：必须用该文件验收
3. ❌ **不跳过统计检查**：必须检查 input_summary、parsing_summary、filter_summary

---

## 10. 版本历史

- **v0.1** (2026-05-15)：初始实现规格

---

**规格状态**：✅ 完成  
**下一步**：等待 CEO 审批  
**维护者**：千尘（CEO）