# Day 9: 状态持久化 — 让 Agent 记住自己的进度

[← Previous: Day 8: 工具调用验证与安全防护](./day-08-工具调用验证与安全防护.md) | [课程目录](../README.md) | [Next: Day 10: Capstone — 设计你的第一个 Agent Harness 架构 →](./day-10-capstone.md)

---

## 🎯 Learning Objectives

By the end of today, you will be able to：

1. **解释** 为什么 JSON 是比 Markdown 更可靠的状态存储格式
2. **实现** 一个简单的检查点保存和恢复机制
3. **描述** Claude Code 如何用 claude-progress.txt + Git 实现状态管理
4. **理解** Git 分支在多 Agent 并行工作中的角色

**Difficulty:** 🟡 Intermediate
**Estimated Time:** 30 minutes

---

## 📚 Core Concepts

### 9.1 状态持久化的核心问题

LLM 是无状态的。但 Agent 工作是有状态的——有"进度"概念，有"做到哪了"。

状态持久化的核心挑战是：**Agent 用自然语言工作，但外部状态存储需要结构化数据。**

自然语言 → LLM → 自然语言（顺畅）
结构化数据 → LLM → 结构化数据（需要约束）

### 9.2 JSON vs Markdown：哪种格式更适合状态存储？

| 维度 | JSON | Markdown |
|------|------|----------|
| 结构化程度 | 高（强类型） | 低（文本为主） |
| LLM 解析可靠性 | 高（格式固定） | 低（模型可能修改格式） |
| 可读性 | 中（对开发者友好） | 高（对人类友好） |
| 机器验证 | 容易（JSON Schema） | 困难（需要 NLP） |
| 嵌套数据 | 支持 | 有限 |

**研究发现：** LLM 破坏 JSON 格式化的概率**显著低于**破坏 Markdown。这意味着用 JSON 做状态存储，Agent 自身的"修改状态"行为更可靠。

### 9.3 状态持久化的三种模式

**模式 1：Progress File（进度文件）**

Claude Code 使用的模式：项目根目录下有一个 `claude-progress.txt` 文件，记录当前进度。

```
当前任务：优化数据库查询性能
上次完成：Step 5 - 识别慢查询
下一步：执行 EXPLAIN 分析
待办：
- [ ] 为 slow_query_2024.sql 添加索引
- [x] 识别出 3 个需要优化的查询
- [ ] 验证优化效果
```

**模式 2：JSON State Store**

结构化更强，机器可读性好：

```json
{
  "task_id": "db-optimization-2024",
  "current_phase": "index-creation",
  "completed_steps": [
    {"step": 1, "action": "identify-slow-queries", "result": "3 queries found"},
    {"step": 2, "action": "analyze-query-plans", "result": "no index on join column"}
  ],
  "pending_steps": [
    {"step": 3, "action": "create-index", "target_table": "orders"},
    {"step": 4, "action": "verify-performance", "expected_improvement": "30%"}
  ],
  "last_updated": "2024-11-15T10:30:00+08:00"
}
```

**模式 3：Git Integration**

Anthropic 的 Claude Code harness 还利用了 Git：
- 每次提交 = 一次检查点
- `git log` = Agent 的任务历史
- `git diff` = Agent 的变更记录
- 分支 = 多任务并行工作空间

> **Key Insight:** 最有效的状态持久化方案，通常是"简单的文件 + 定期保存"，而非复杂的数据库设计。Claude Code 的实践证明了这一点——`claude-progress.txt` + `git log` 是 Anthropic 内部最广泛使用的状态管理工具组合。

---

## 🔑 Key Terminology

| Term | Definition | Example |
|------|-----------|---------|
| **Progress File** | 记录 Agent 当前进度的文本文件，通常为 Claude Code 的 progress.txt 格式 | claude-progress.txt |
| **JSON State Store** | 用 JSON 格式存储的结构化 Agent 状态，可被 LLM 可靠读写 | 包含 completed_steps、pending_steps 的 JSON 对象 |
| **Git Integration** | 利用 Git 作为 Agent 任务的版本控制系统，记录变更历史 | git commit = 检查点，git log = 任务历史 |

---

## 💻 Code Examples

### Example: 轻量级 JSON State Store

```python
# -*- coding: utf-8 -*-
"""
轻量级 JSON State Store — 让 Agent 可靠地读写自己的进度
"""

import json
import os
from datetime import datetime

class SimpleStateStore:
    """最简单的 JSON 状态存储，Agent 可以在其中读写"""
    
    def __init__(self, filepath="agent_state.json"):
        self.filepath = filepath
        if os.path.exists(filepath):
            with open(filepath) as f:
                self.state = json.load(f)
        else:
            self.state = self._blank_state()
    
    def _blank_state(self):
        return {
            "created_at": datetime.now().isoformat(),
            "task": None,
            "phase": "init",
            "completed": [],
            "pending": [],
            "artifacts": {}
        }
    
    # Agent 可以调用这些方法
    def set_task(self, task_name):
        self.state["task"] = task_name
        self._save()
    
    def add_completed(self, step_id, action, result):
        self.state["completed"].append({
            "step": step_id,
            "action": action,
            "result": result,
            "completed_at": datetime.now().isoformat()
        })
        self._save()
    
    def get_status(self):
        return {
            "task": self.state["task"],
            "progress": f"{len(self.state['completed'])}/{len(self.state['completed']) + len(self.state['pending'])} steps done",
            "last_action": self.state["completed"][-1]["action"] if self.state["completed"] else "none"
        }
    
    def _save(self):
        with open(self.filepath, 'w') as f:
            json.dump(self.state, f, indent=2)

# Agent 使用示例：
store = SimpleStateStore("my_agent_state.json")
store.set_task("优化数据库查询")
print(store.get_status())
```

---

## ✏️ Hands-On Exercises

### Exercise 1: 设计你的状态模式（⏱️ ~15 min）

**Goal:** 为一个自动化报表生成 Agent 设计状态持久化方案

**Tasks:**
1. 列出 Agent 需要跟踪的所有状态信息
2. 选择 JSON / Markdown / Git 之一作为存储格式
3. 设计具体的状态结构

---

### Exercise 2: Claude Code 的 Progress File 模式（⏱️ ~10 min）

**Goal:** 理解 Claude Code 如何用 progress.txt 工作

**Instructions:** 思考如果你来实现一个类似机制，你的 progress.txt 格式会是什么样的？与 Claude Code 的格式对比有什么优劣？

---

## ➡️ Next Steps

**Tomorrow: Day 10 — Capstone：设计你的第一个 Agent Harness 架构**

[← Previous: Day 8: 工具调用验证与安全防护](./day-08-工具调用验证与安全防护.md) | [课程目录](../README.md) | [Next: Day 10: Capstone →](./day-10-capstone.md)
