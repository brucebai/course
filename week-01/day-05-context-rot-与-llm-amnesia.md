# Day 5: Context Rot 与 LLM Amnesia — 被遗忘的上下文

[← Previous: Day 4: Long-Running Agents 三大失败原因](./day-04-long-running-agents-三大失败原因.md) | [课程目录](../README.md) | [Next: Day 6: Harness 的六大核心组件概览 →](../week-02/day-06-六大核心组件概览.md)

---

## 🎯 Learning Objectives

By the end of today, you will be able to：

1. **解释** Context Rot（上下文腐烂）的机制和影响
2. **描述** LLM Amnesia（健忘症）的表现和危害
3. **理解** 为什么"有状态"和"无状态"对 Agent 如此关键
4. **识别** 生产环境中 Context Rot 的早期预警信号
5. **了解** Harness 通过哪些机制对抗 Context Rot 和 Amnesia

**Difficulty:** 🟡 Intermediate
**Estimated Time:** 30 minutes

---

## 📚 Core Concepts

### 5.1 Context Rot：沉默的杀手

**Context Rot（上下文腐烂）**是一个慢性的、累积的过程。与其说它是某个时刻的"故障"，不如说它是 Long-Running Agent 的**默认状态**。

**发生了什么？**

每个 AI 模型都有一个上下文窗口（Context Window），这是它一次能处理的最大 token 数。当窗口快满时，系统必须做出选择：哪些信息保留，哪些丢弃。

通常的策略是**先入先出（FIFO）**：最近的信息保留，最早的信息丢弃。

这在大多数对话中没问题，因为：
- 用户的请求简短
- 任务在几次交互内完成
- 最重要的信息（最新回复）总是被保留

但在 Long-Running Agent 中，FIFO 策略是**灾难性的**，因为：

**最重要的信息往往是"最初的指令"，而不是"最新的输出"。**

```
第 1 步：用户说"帮我把这 5000 条客户记录去重，邮箱相同样本保留最新一条"
  → 原始指令：5000 条记录去重，规则：邮箱重复保留最新

第 10 步：处理了 1000 条记录，输出了一堆中间状态
  → 原始指令还在，但已经被推到了窗口边缘

第 30 步：处理到 3000 条，开始出现异常情况
  → Agent 需要知道"规则是保留最新一条，邮箱是唯一标识"
  → 但原始指令已经非常模糊，Agent 开始"自作主张"

第 50 步：完成了，但结果完全错误
  → Agent 用自己的理解替代了原始指令（"去重"变成了"随机删除"）
```

这就是 Context Rot。它不是突然发生的，而是**每一步都在积累，每一步都在腐蚀**。当团队最终发现输出结果不符合预期时，原因已经很难追溯了。

### 5.2 LLM Amnesia：断点后的空白

**LLM Amnesia（健忘症）**指的是：当 Agent 因网络错误、系统崩溃或用户中断而"重启"时，它会丢失在崩溃前积累的所有上下文和工作进度。

**为什么 LLM 会"失忆"？**

大多数 LLM 在架构上是**无状态（Stateless）**的。这意味着：
- 每次发送请求，模型是从零开始的
- 模型不记得任何之前发生的事
- "记忆"来自于每次请求时传入的上下文

如果你的 Agent 崩溃了，但你的 Harness 没有持久化机制，那么：
- 崩溃前 30 分钟的工作全部丢失
- Agent 重启后不知道上次做到哪了
- 必须从头开始——用户不得不重复整个流程

**真实的"健忘症"案例：**

> 某团队的自动化报告生成 Agent 需要 3-4 小时完成一份报告。突然断电或网络中断后，Agent 重启，从头开始，用户等了 4 小时得到一份"重新开始"的确认消息，而原来的进度全部丢失。

这个问题在 LangChain 和 AutoGen 的早期版本中非常普遍，后来才逐渐通过状态持久化机制解决。

### 5.3 为什么 Agent 比普通 Chatbot 更需要 Memory？

普通聊天机器人不需要担心 Long-Running 场景，因为：
- 任务简单：问一个问题，回答一个问题，结束
- 时间短：几秒钟内完成
- 不需要状态：因为任务本身没有"进度"概念

但 Agent 的工作方式完全不同：
- 任务复杂：多步骤、多工具、跨系统
- 时间长：数小时到数天
- 需要状态：任务有"做到哪了"的概念

这就像：
- **短时记忆（Chatbot）：** 记住对方刚才说的话，回答下一句
- **长时记忆（Agent）：** 记住项目的整体目标、中间状态、待办事项

**Context Window ≠ 记忆系统。** 把 Context Window 类比为内存条（RAM）更准确——它是易失性的，断电就没了。而真正需要的，是一个持久化的"硬盘"——外部状态存储。

### 5.4 Harness 如何对抗 Context Rot 和 Amnesia？

| 问题 | Harness 解决方案 | 实现方式 |
|------|-----------------|----------|
| **Context Rot** | 上下文工程（Context Engineering） | 定期压缩、定期刷新原始目标、上下文分区 |
| **Context Rot** | 工具输出 Offloading | 大型工具输出存到外部，不进入 Context Window |
| **LLM Amnesia** | 状态持久化 | 定期将进度写入外部存储（JSON 文件、数据库） |
| **LLM Amnesia** | 断点恢复 | Agent 重启时从外部存储读取状态，无缝衔接 |

> **Key Insight:** Context Window 是 RAM（断电即失），不是硬盘。Context Rot 和 LLM Amnesia 不是模型的 bug，而是使用有限上下文窗口处理无限任务的必然代价。Harness 的任务，就是在这两个限制下，让 Agent 仍然能可靠地完成任务。

---

## 🔑 Key Terminology

| Term | Definition | Example |
|------|-----------|---------|
| **Context Rot（上下文腐烂）** | 关键信息（原始目标）在 Context Window 中逐渐被后续信息淹没的过程 | Agent 在第 30 步忘了第 1 步的指令 |
| **LLM Amnesia（健忘症）** | Agent 重启或中断后丢失所有进度，因为缺乏外部状态存储 | 断电后 Agent 从头开始，上次工作全部丢失 |
| **Context Engineering** | 主动管理 Context Window 内容的实践，包括压缩、刷新、分区 | Claude Agent SDK 的自动压缩功能 |
| **Stateless（无状态）** | 模型架构特性：每次请求独立，无持久"记忆" | 大多数 LLM API 的工作方式 |
| **Tool Output Offloading** | 将大型工具输出存到外部存储，不注入 Context Window 的技术 | 搜索 API 返回 5000 字结果，直接存文件而不是注入上下文 |
| **Checkpoint（检查点）** | Agent 在特定步骤保存的工作状态快照 | Claude Code 的 claude-progress.txt |

---

## 💻 Code Examples

### Example 1: 简单的状态持久化实现

```python
# -*- coding: utf-8 -*-
"""
演示：Agent 的状态持久化 — 让 Agent"记住"上次做到哪了
"""

import json
import os
from datetime import datetime

class AgentStateStore:
    """简单的 JSON 文件状态存储"""
    
    def __init__(self, task_name, state_file="agent_state.json"):
        self.state_file = state_file
        self.task_name = task_name
        self.state = self._load_or_init()
    
    def _load_or_init(self):
        """从文件加载状态，或初始化新状态"""
        if os.path.exists(self.state_file):
            with open(self.state_file, 'r') as f:
                return json.load(f)
        else:
            return {
                "task_name": self.task_name,
                "created_at": datetime.now().isoformat(),
                "last_updated": datetime.now().isoformat(),
                "completed_steps": [],
                "current_step": 0,
                "progress_percent": 0,
                "artifacts": {}  # 中间结果存储
            }
    
    def save(self):
        """保存当前状态到文件"""
        self.state["last_updated"] = datetime.now().isoformat()
        with open(self.state_file, 'w') as f:
            json.dump(self.state, f, indent=2, ensure_ascii=False)
        print(f"💾 状态已保存：Step {self.state['current_step']}, {self.state['progress_percent']}%")
    
    def update_step(self, step, result=None):
        """更新步骤记录"""
        self.state["current_step"] = step
        if step not in self.state["completed_steps"]:
            self.state["completed_steps"].append(step)
        if result:
            self.state["artifacts"][f"step_{step}"] = result
        self.state["progress_percent"] = min(step * 10, 100)
        self.save()
    
    def load(self):
        """模拟 Agent 重启后加载状态"""
        return self._load_or_init()


# 演示：正常运行 + 崩溃 + 恢复
print("=" * 55)
print("演示：状态持久化保护 Agent 免于 LLM Amnesia")
print("=" * 55)

# 模拟 Agent 运行
store = AgentStateStore("客户数据去重任务")

print("\n📍 Step 1-5: Agent 正常运行中...")
for step in range(1, 6):
    store.update_step(step, {"records_processed": step * 100})

print("\n💥 模拟系统崩溃！断电！Agent 重启...")
print("（没有状态持久化的情况下，这里 Agent 会丢失所有进度）")

# 模拟 Agent 重启后恢复
print("\n🔄 Agent 重启后，从外部存储恢复状态...")
restored = AgentStateStore("客户数据去重任务")
state = store.load()
print(f"✅ 恢复成功！")
print(f"   当前进度：Step {state['current_step']}/10")
print(f"   已完成步骤：{state['completed_steps']}")
print(f"   上次保存时间：{state['last_updated']}")

# 继续从断点执行
print("\n▶️  Agent 从 Step 6 继续执行（不从头开始）...")
for step in range(6, 11):
    store.update_step(step, {"records_processed": step * 100})
```

**What's happening:** 这个示例展示了状态持久化的基本模式：定期将 Agent 进度保存到外部 JSON 文件，Agent 重启时从文件恢复而不是从头开始。这是对抗 LLM Amnesia 的核心机制。

---

## ✏️ Hands-On Exercises

### Exercise 1: 症状识别（⏱️ ~10 min）

**Goal:** 从日志中识别 Context Rot 症状

**Instructions:** 阅读以下 Agent 日志片段，找出 Context Rot 的迹象

```
[Step 1]  Agent: 用户请求"帮我把这1000条订单按状态分类，pending=发邮件提醒，completed=归档"
[Step 5]  Agent: 正在处理第500条，已分类 pending: 120, completed: 380
[Step 10] Agent: 注意到有重复邮箱，正在去重处理
[Step 15] Agent: 开始对数据进行规范化处理，统一日期格式
[Step 20] Agent: 输出结果文件...但用户原始请求是"分类"，不是"去重"
```

**Questions:**
1. 在 Step 20 时，Agent 实际上在执行什么操作？
2. 这个操作是否在用户的原始请求中？
3. Context Rot 在这里是如何体现的？

---

### Exercise 2: 设计你的状态持久化方案（⏱️ ~15 min）

**Goal:** 为一个具体的 Agent 任务设计状态持久化方案

**Scenario:** 一个自动化价格监控 Agent，每小时检查一次某商品价格，价格跌破阈值时发邮件通知

**Questions:**
1. 这个 Agent 需要持久化哪些状态？（提示：至少考虑 3 个维度）
2. 如果 Agent 在凌晨 3 点崩溃，理想情况下重启后应该从哪一步恢复？
3. 你的方案中，检查点应该设置在哪些位置？

---

### Exercise 3: 计算题（⏱️ ~5 min）

**Goal:** 计算 Context Window 消耗

**Question:** 一个 Agent 的 Context Window 是 200K tokens。原始指令 1000 tokens，每步工具输出平均 5000 tokens。如果 Agent 运行 40 步后，Context Window 已经满了。问：
- 总共消耗了多少 tokens？（不含历史累计）
- 原始指令还在窗口里吗？

---

## 📖 Curated Resources

### Must-Read

1. **The Problem of Context Rot** — MemU 2026 Research
   - 🔗 可搜索 "MemU context retention loss per step LLM"
   - Why: 学术数据：2%/步的上下文保留衰减率

2. **Claude Agent SDK 的上下文压缩** — Anthropic 官方文档
   - 🔗 https://docs.anthropic.com/ （搜索 Claude Agent SDK）
   - Why: 了解业界领先的上下文管理实践

---

## 🤔 Reflection Questions

1. **Comprehension:** 用一句话解释 Context Rot 和 LLM Amnesia 的本质区别。

2. **Application:** 你当前使用的 AI 工具（ChatGPT、Claude 等）在长对话中是否有 Context Rot 的迹象？你是如何应对的？

3. **Critical Thinking:** 如果给你一个无限 Context Window 的 LLM，LLM Amnesia 还会存在吗？为什么？

---

## ➡️ Next Steps

**Tomorrow: Day 6 — Harness 的六大核心组件概览**

第一周的学习到此结束！今天我们建立了基础：什么是 Agent Harness、为什么它重要、三大失败原因、以及 Context Rot/Amnesia 的机制。

**Week 2 预告：** 从明天开始，我们将进入"Building"阶段，深入学习 Harness 的六大核心组件，并动手设计你的第一个 Agent Harness 架构。

**Before moving on, make sure you can：**
- [ ] 解释 Context Rot 和 LLM Amnesia 的机制差异
- [ ] 说出至少 2 种 Harness 对抗 Context Rot 的策略
- [ ] 描述状态持久化的基本实现方式

[← Previous: Day 4: Long-Running Agents 三大失败原因](./day-04-long-running-agents-三大失败原因.md) | [课程目录](../README.md) | [Next: Day 6: Harness 的六大核心组件概览 →](../week-02/day-06-六大核心组件概览.md)
