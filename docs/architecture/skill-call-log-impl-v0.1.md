# Skill 调用日志 最小实现方案 v0.1

> **核心目标：只做日志写入模板，不改变现有行为**
> 
> 只实现本地日志 + 公共脱敏日志的写入模板。
> 不改变 ceo 行为，不接任务队列，不接 Codex Runner，不接 Telegram 回调，不自动执行。

**版本：** v0.1  
**日期：** 2026-05-13  
**作者：** 千尘 🌌  
**状态：** 方案，待确认  
**前置：** skill-call-log-v0.3 设计通过

---

## 一、实现原则

### 1.1 只做写入模板

**核心原则：** 只提供日志写入的模板和工具，不改变 ceo 的现有行为。

**做的事：**
- ✅ 提供日志写入函数
- ✅ 提供脱敏函数
- ✅ 提供日志文件管理
- ✅ 提供示例模板

**不做的事：**
- ❌ 不改变 ceo 的派工方式
- ❌ 不改变 ceo 的汇报方式
- ❌ 不接任务队列
- ❌ 不接 Codex Runner
- ❌ 不接 Telegram 回调
- ❌ 不自动执行

### 1.2 双日志架构

**本地完整日志：**
- 位置：`ai-company/skill-call-logs/YYYY-MM-DD.jsonl`
- 内容：完整记录，包含所有字段
- 用途：ceo 内部追溯、调试、审计
- 安全：仅本机存储，不推送

**公共脱敏日志：**
- 位置：`qianchen-os-docs/task-logs/public/skill-calls/YYYY-MM-DD.jsonl`
- 内容：脱敏后记录，仅保留必要信息
- 用途：公开审计、协作、分析
- 安全：脱敏后推送，无敏感信息

### 1.3 手动触发

**ceo 在决策时手动调用：**
- 不自动触发
- 不自动检测
- 不自动记录

---

## 二、文件结构

### 2.1 目录结构

```
ai-company/
├── skill-call-logs/              # 本地完整日志
│   ├── 2026-05-13.jsonl
│   ├── 2026-05-14.jsonl
│   └── ...
├── skill_logger.py               # 日志写入模块
└── skill_desensitizer.py         # 脱敏模块

qianchen-os-docs/
└── task-logs/
    └── public/
        └── skill-calls/          # 公共脱敏日志
            ├── 2026-05-13.jsonl
            ├── 2026-05-14.jsonl
            └── ...
```

### 2.2 文件命名

**本地日志：**
- 格式：`YYYY-MM-DD.jsonl`
- 示例：`2026-05-13.jsonl`

**公共日志：**
- 格式：`YYYY-MM-DD.jsonl`
- 示例：`2026-05-13.jsonl`

---

## 三、日志写入模块

### 3.1 模块功能

**skill_logger.py：**
- 提供日志写入函数
- 管理日志文件
- 提供示例模板

### 3.2 核心函数

**log_skill_call()：**
```python
def log_skill_call(
    user_message: str,
    task_description: str,
    task_category: str,
    channel_type: str,
    skill_check: dict,
    async_check: dict,
    parallel_check: dict,
    codex_check: dict,
    approval_check: dict,
    decision: str,
    decision_detail: str = None,
    action: str = None,
    task_id: str = None
) -> str:
    """
    记录 skill 调用日志
    
    返回：日志文件路径
    """
    # 构建日志条目
    log_entry = {
        "timestamp": get_timestamp(),
        "session_id": get_session_id(),
        "channel_type": channel_type,
        "user_message": user_message,
        "task_description": task_description,
        "task_category": task_category,
        "skill_check": skill_check,
        "async_check": async_check,
        "parallel_check": parallel_check,
        "codex_check": codex_check,
        "approval_check": approval_check,
        "decision": decision,
        "decision_detail": decision_detail,
        "action": action,
        "task_id": task_id
    }
    
    # 写入本地日志
    log_file = append_to_local_log(log_entry)
    
    return log_file
```

**append_to_local_log()：**
```python
def append_to_local_log(log_entry: dict) -> str:
    """
    追加到本地日志文件
    
    返回：日志文件路径
    """
    # 获取日期
    date_str = log_entry["timestamp"][:10]
    
    # 构建文件路径
    log_dir = "ai-company/skill-call-logs"
    os.makedirs(log_dir, exist_ok=True)
    log_file = os.path.join(log_dir, f"{date_str}.jsonl")
    
    # 追加日志
    with open(log_file, "a", encoding="utf-8") as f:
        f.write(json.dumps(log_entry, ensure_ascii=False) + "\n")
    
    return log_file
```

### 3.3 使用示例

**ceo 调用：**
```python
from skill_logger import log_skill_call

# 记录日志
log_file = log_skill_call(
    user_message="检查一下 memory 状态",
    task_description="检查V5记忆系统健康状态",
    task_category="系统检查",
    channel_type="TELEGRAM",
    skill_check={
        "has_skill": True,
        "skill_name": "codex.check_v5",
        "skill_path": "skills/codex-check-v5",
        "skill_matched": True,
        "skill_called": False,
        "not_called_reason": "ASYNC_REQUIRED"
    },
    async_check={
        "need_async": True,
        "async_reason": "预计 > 10秒",
        "estimated_duration": 30
    },
    parallel_check={
        "need_parallel": False,
        "parallel_reason": "单一检查任务",
        "parallel_tasks": []
    },
    codex_check={
        "need_codex": True,
        "codex_reason": "需要检查文件系统",
        "codex_mode": "readonly"
    },
    approval_check={
        "need_human_approval": False,
        "approval_reason": "只读检查，低风险",
        "risk_level": "LOW"
    },
    decision="ASYNC_DISPATCH",
    decision_detail="异步派工给 coding_agent + Codex",
    action="create_task",
    task_id="task-20260513-001"
)

print(f"日志已记录到：{log_file}")
```

---

## 四、脱敏模块

### 4.1 模块功能

**skill_desensitizer.py：**
- 提供脱敏函数
- 生成公共日志
- 管理脱敏规则

### 4.2 核心函数

**desensitize_log_entry()：**
```python
def desensitize_log_entry(local_log_entry: dict) -> dict:
    """
    脱敏本地日志条目
    
    返回：脱敏后的公共日志条目
    """
    # 提取必要字段
    public_log_entry = {
        "timestamp": local_log_entry["timestamp"],
        "channel_type": local_log_entry["channel_type"],
        "task_category": local_log_entry["task_category"],
        "skill_name": local_log_entry["skill_check"]["skill_name"],
        "skill_matched": local_log_entry["skill_check"]["skill_matched"],
        "skill_called": local_log_entry["skill_check"]["skill_called"],
        "not_called_reason": local_log_entry["skill_check"]["not_called_reason"],
        "need_async": local_log_entry["async_check"]["need_async"],
        "need_parallel": local_log_entry["parallel_check"]["need_parallel"],
        "need_codex": local_log_entry["codex_check"]["need_codex"],
        "need_human_approval": local_log_entry["approval_check"]["need_human_approval"],
        "risk_level": local_log_entry["approval_check"]["risk_level"],
        "decision": local_log_entry["decision"]
    }
    
    return public_log_entry
```

**generate_public_log()：**
```python
def generate_public_log(date_str: str = None) -> str:
    """
    生成公共脱敏日志
    
    参数：
        date_str: 日期字符串，格式 YYYY-MM-DD，默认今天
    
    返回：公共日志文件路径
    """
    # 默认今天
    if date_str is None:
        date_str = datetime.now().strftime("%Y-%m-%d")
    
    # 本地日志路径
    local_log_file = f"ai-company/skill-call-logs/{date_str}.jsonl"
    
    # 公共日志路径
    public_log_dir = "qianchen-os-docs/task-logs/public/skill-calls"
    os.makedirs(public_log_dir, exist_ok=True)
    public_log_file = os.path.join(public_log_dir, f"{date_str}.jsonl")
    
    # 读取本地日志
    if not os.path.exists(local_log_file):
        print(f"本地日志不存在：{local_log_file}")
        return None
    
    # 脱敏并写入公共日志
    with open(local_log_file, "r", encoding="utf-8") as f_in:
        with open(public_log_file, "w", encoding="utf-8") as f_out:
            for line in f_in:
                local_entry = json.loads(line)
                public_entry = desensitize_log_entry(local_entry)
                f_out.write(json.dumps(public_entry, ensure_ascii=False) + "\n")
    
    print(f"公共日志已生成：{public_log_file}")
    return public_log_file
```

### 4.3 使用示例

**ceo 手动调用：**
```python
from skill_desensitizer import generate_public_log

# 生成今天的公共日志
public_log_file = generate_public_log()

# 生成指定日期的公共日志
public_log_file = generate_public_log("2026-05-13")

print(f"公共日志已生成到：{public_log_file}")
```

---

## 五、枚举定义

### 5.1 not_called_reason 枚举

```python
class NotCalledReason(Enum):
    """skill 未调用原因"""
    NO_MATCH = "NO_MATCH"                    # 没有匹配的 skill
    ASYNC_REQUIRED = "ASYNC_REQUIRED"        # 任务需要异步，skill 不支持
    SEQUENTIAL_TASK = "SEQUENTIAL_TASK"      # 串行任务，不适合并行
    CONTEXT_REQUIRED = "CONTEXT_REQUIRED"    # 需要完整上下文
    SHARED_STATE = "SHARED_STATE"            # 有共享状态，不适合并行
    LOW_PRIORITY = "LOW_PRIORITY"            # 低优先级，暂不处理
    MANUAL_APPROVAL_REQUIRED = "MANUAL_APPROVAL_REQUIRED"  # 需要人工确认
    SKILL_DEPRECATED = "SKILL_DEPRECATED"    # skill 已废弃
    OTHER = "OTHER"                          # 其他原因
```

### 5.2 decision 枚举

```python
class Decision(Enum):
    """最终决策"""
    SYNC_EXECUTE = "SYNC_EXECUTE"            # 同步执行
    ASYNC_DISPATCH = "ASYNC_DISPATCH"        # 异步派工
    PARALLEL_DISPATCH = "PARALLEL_DISPATCH"  # 并行派工
    HUMAN_APPROVAL = "HUMAN_APPROVAL"        # 等待人工确认
    SKIP = "SKIP"                            # 跳过，不执行
    DEFER = "DEFER"                          # 延后处理
    ESCALATE = "ESCALATE"                    # 升级处理
```

### 5.3 channel_type 枚举

```python
class ChannelType(Enum):
    """渠道类型"""
    TELEGRAM = "TELEGRAM"                    # Telegram 渠道
    DIRECT = "DIRECT"                        # 直连（OpenClaw 主会话）
    OFFICE_3D = "3D_OFFICE"                  # 3D 办公室
    APP = "APP"                              # 未来 App
    CRON = "CRON"                            # 定时任务
    SYSTEM = "SYSTEM"                        # 系统触发
```

### 5.4 risk_level 枚举

```python
class RiskLevel(Enum):
    """风险等级"""
    LOW = "LOW"                              # 低风险（只读、检查）
    MEDIUM = "MEDIUM"                        # 中风险（写入、修改）
    HIGH = "HIGH"                            # 高风险（删除、发布）
    CRITICAL = "CRITICAL"                    # 极高风险（系统配置、密钥）
```

---

## 六、任务类别映射

### 6.1 常见任务类别

```python
TASK_CATEGORY_MAP = {
    # 系统检查类
    "检查": "系统检查",
    "check": "系统检查",
    "诊断": "系统检查",
    "diagnose": "系统检查",
    
    # bug 修复类
    "修复": "bug 修复",
    "fix": "bug 修复",
    "修复bug": "bug 修复",
    
    # 代码修改类
    "修改": "代码修改",
    "修改代码": "代码修改",
    "重构": "代码修改",
    "refactor": "代码修改",
    
    # 批量检查类
    "同时检查": "批量检查",
    "批量检查": "批量检查",
    "多个系统": "批量检查",
    
    # 架构设计类
    "设计": "架构设计",
    "架构": "架构设计",
    "architecture": "架构设计",
    
    # 文档编写类
    "编写": "文档编写",
    "写文档": "文档编写",
    "README": "文档编写",
    
    # 测试执行类
    "测试": "测试执行",
    "运行测试": "测试执行",
    "test": "测试执行",
    
    # 部署发布类
    "部署": "部署发布",
    "发布": "部署发布",
    "deploy": "部署发布",
    
    # 监控告警类
    "监控": "监控告警",
    "告警": "监控告警",
    "alert": "监控告警",
}

def categorize_task(task_description: str) -> str:
    """
    将任务描述分类为任务类别
    
    返回：任务类别
    """
    task_lower = task_description.lower()
    
    for keyword, category in TASK_CATEGORY_MAP.items():
        if keyword.lower() in task_lower:
            return category
    
    return "其他"
```

---

## 七、目录创建

### 7.1 创建目录脚本

**create_log_dirs.py：**
```python
#!/usr/bin/env python3
"""
创建日志目录结构
"""

import os

def create_log_dirs():
    """创建日志目录"""
    dirs = [
        "ai-company/skill-call-logs",
        "qianchen-os-docs/task-logs/public/skill-calls"
    ]
    
    for dir_path in dirs:
        os.makedirs(dir_path, exist_ok=True)
        print(f"目录已创建：{dir_path}")

if __name__ == "__main__":
    create_log_dirs()
```

### 7.2 运行脚本

```bash
python3 create_log_dirs.py
```

---

## 八、使用流程

### 8.1 ceo 记录日志

**步骤：**
1. ceo 做出任务决策
2. 调用 `log_skill_call()` 记录本地日志
3. 继续执行原有逻辑

**示例：**
```python
# ceo 决策后
log_skill_call(
    user_message="检查一下 memory 状态",
    task_description="检查V5记忆系统健康状态",
    task_category="系统检查",
    channel_type="TELEGRAM",
    skill_check={...},
    async_check={...},
    parallel_check={...},
    codex_check={...},
    approval_check={...},
    decision="ASYNC_DISPATCH"
)

# 继续原有逻辑
# ...
```

### 8.2 生成公共日志

**步骤：**
1. 定期（如每天）调用 `generate_public_log()`
2. 生成脱敏后的公共日志
3. 手动推送到公共仓库

**示例：**
```python
# 生成今天的公共日志
public_log_file = generate_public_log()

# 生成指定日期的公共日志
public_log_file = generate_public_log("2026-05-13")

# 手动推送到公共仓库
# git add qianchen-os-docs/task-logs/public/skill-calls/
# git commit -m "更新 skill 调用日志"
# git push origin main
```

---

## 九、安全规则

### 9.1 本地日志安全

**规则：**
- 本地日志包含完整信息，仅本机存储
- 本地日志绝不自动推送公共仓库
- 本地日志可用于内部追溯和调试

### 9.2 公共日志安全

**规则：**
- 公共日志必须经过脱敏函数生成
- 公共日志不包含敏感信息
- 公共日志可安全推送到公共仓库

### 9.3 脱敏规则

**必须删除的字段：**
- `user_message` 原文
- `session_id`
- `decision_detail`
- 任何包含人名、账号、路径、ID 的字段

**必须保留的字段：**
- `timestamp`
- `channel_type`
- `task_category`
- `skill_name`
- `skill_matched`
- `skill_called`
- `not_called_reason`（枚举）
- `need_async`
- `need_parallel`
- `need_codex`
- `need_human_approval`
- `risk_level`（枚举）
- `decision`（枚举）

---

## 十、测试用例

### 10.1 测试本地日志写入

```python
from skill_logger import log_skill_call

# 测试用例 1：系统检查
log_file = log_skill_call(
    user_message="检查一下 memory 状态",
    task_description="检查V5记忆系统健康状态",
    task_category="系统检查",
    channel_type="TELEGRAM",
    skill_check={
        "has_skill": True,
        "skill_name": "codex.check_v5",
        "skill_path": "skills/codex-check-v5",
        "skill_matched": True,
        "skill_called": False,
        "not_called_reason": "ASYNC_REQUIRED"
    },
    async_check={
        "need_async": True,
        "async_reason": "预计 > 10秒",
        "estimated_duration": 30
    },
    parallel_check={
        "need_parallel": False,
        "parallel_reason": "单一检查任务",
        "parallel_tasks": []
    },
    codex_check={
        "need_codex": True,
        "codex_reason": "需要检查文件系统",
        "codex_mode": "readonly"
    },
    approval_check={
        "need_human_approval": False,
        "approval_reason": "只读检查，低风险",
        "risk_level": "LOW"
    },
    decision="ASYNC_DISPATCH",
    decision_detail="异步派工给 coding_agent + Codex",
    action="create_task",
    task_id="task-20260513-001"
)

print(f"测试用例 1 完成：{log_file}")
```

### 10.2 测试公共日志生成

```python
from skill_desensitizer import generate_public_log

# 测试用例 1：生成今天的公共日志
public_log_file = generate_public_log()
print(f"测试用例 1 完成：{public_log_file}")

# 测试用例 2：生成指定日期的公共日志
public_log_file = generate_public_log("2026-05-13")
print(f"测试用例 2 完成：{public_log_file}")
```

---

## 十一、部署步骤

### 11.1 创建目录

```bash
python3 create_log_dirs.py
```

### 11.2 部署模块

**将模块放到 ai-company 目录：**
```bash
cp skill_logger.py ai-company/
cp skill_desensitizer.py ai-company/
cp create_log_dirs.py ai-company/
```

### 11.3 测试

```bash
cd ai-company
python3 -c "from skill_logger import log_skill_call; print('模块导入成功')"
python3 -c "from skill_desensitizer import generate_public_log; print('模块导入成功')"
```

---

## 十二、注意事项

### 12.1 不改变现有行为

**ceo 继续按现有方式工作：**
- 继续识别任务
- 继续判断是否使用 skill
- 继续派工
- 继续汇报

**只添加记录：**
- 在决策时调用 `log_skill_call()`
- 不改变决策过程
- 不改变执行方式

### 12.2 手动触发

**ceo 手动调用：**
- 不自动触发
- 不自动检测
- 不自动记录

### 12.3 安全第一

**本地日志：**
- 包含完整信息
- 仅本机存储
- 绝不自动推送

**公共日志：**
- 必须脱敏
- 不包含敏感信息
- 手动推送

---

## 十三、下一步

### 13.1 方案确认

1. **设计通过** - 等待雨陌确认
2. **代码实现** - 实现日志模块
3. **测试验证** - 测试日志写入和脱敏

### 13.2 部署上线

1. **部署模块** - 将模块放到 ai-company 目录
2. **创建目录** - 创建日志目录
3. **测试运行** - 测试模块功能

### 13.3 使用推广

1. **ceo 培训** - 培训 ceo 使用日志模块
2. **定期审计** - 定期检查日志
3. **持续优化** - 根据日志优化触发条件

---

*实现方案 — 2026-05-13*
