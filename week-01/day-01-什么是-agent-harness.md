# Day 1: 什么是 Agent Harness？— 从 AI 模型到企业级 Agent

[← 课程目录](../README.md) | [Next: Day 2: Agent vs Harness →](./day-02-agent-vs-harness.md)

---

## 🎯 Learning Objectives

By the end of today, you will be able to：

1. **解释** Agent Harness 的定义及其在 AI Agent 系统中的位置
2. **描述** AI 模型单独存在时的局限性
3. **理解** 为什么"模型足够好"不等于"产品足够好"
4. **识别** Agent 在生产环境中遭遇的典型失败场景
5. **说明** Harness 作为"翻译层"的核心价值

**Difficulty:** 🟢 Beginner
**Estimated Time:** 30 minutes

---

## 📚 Core Concepts

### 1.1 从"会聊天"到"能干活"

过去两年，生成式 AI（Generative AI）的飞速进展让人们对大语言模型（LLM）印象深刻——文本生成、摘要、逻辑推理、数学计算，似乎已经无所不能。

但当企业真正把生成式 AI 部署到业务流程中时，情况变了。这些企业发现：

> **一个模型本身不是产品。**（A model on its own is not a product.）

在演示环境中，AI Agent 只需要处理单一任务——回答一个问题、生成一段文字。而在生产环境中，一个 Agent 可能需要：
- 调用外部 API 获取数据
- 访问数据库读写记录
- 跨多个工具协作完成任务
- 在数小时甚至数天内持续运行

模型在训练时见过的，是独立的任务片段。模型没有见过的，是真实生产环境中的**复杂性、不确定性和长时运行**。

### 1.2 模型的四大约束

即使是最先进的 LLM，当它**独自面对生产环境**时，会遇到四大约束：

| 约束 | 描述 | 示例 |
|------|------|------|
| **API 超时** | 外部 API 无响应时，模型不知道该等待还是重试 | 支付接口响应慢，Agent 就卡死 |
| **内存耗尽** | 对话上下文窗口有上限，Agent 无法记住长期目标 | 任务跑了 100 步，早期目标已被淹没 |
| **工具乱序** | 模型可能以错误顺序调用工具，或遗漏必要步骤 | 先删数据，后查记录 |
| **函数幻觉** | 模型可能调用一个不存在的函数，或传错误的参数 | "我已经调用了 xxx 接口"（其实没有） |

这些问题，**不是模型不够聪明**，而是模型缺乏一个"执行环境"来管理它与外部世界的交互。

### 1.3 Agent Harness 的定义

**Agent Harness**（有时简称 harness）是包裹在 AI 模型外层的**软件基础设施层**，它负责：

- **生命周期管理**（Lifecycle Management）：Agent 何时启动、暂停、恢复、终止
- **上下文管理**（Context Management）：什么信息进入模型的上下文窗口，什么被归档
- **工具访问**（Tool Access）：管理 Agent 可以调用的工具集，处理工具的输入输出
- **安全防护**（Safety & Guardrails）：防止 Agent 执行危险操作，设置权限边界
- **状态持久化**（State Persistence）：在会话之间保存进度，断点恢复

更直观地理解：想象 AI 模型是一匹野马——力量强大，但没有方向感，不懂边界，不知道"停"。**Harness 就是缰绳、笼头和马鞍**，把野马的力量引导成可控的工作。

### 1.4 Harness 不等于 Framework

这里有一个常见误区：Harness 和 Framework（框架）不是一回事。

| | Framework（框架） | Harness（工具架） |
|--|---|---|
| **角色** | 蓝图/组件库 | 运行时执行环境 |
| **提供物** | 组件库，帮你"组装"Agent | 运行时governance，管理 Agent 的行为 |
| **类比** | 乐高积木 | 乐高手册 + 工作台 + 装配流程 |
| **示例** | LangChain、AutoGen | Claude Code 底层架构、Salesforce Agentforce |

Framework 给你组件，Harness 给你运行时。一个帮助你**构建** Agent，一个帮助 Agent **可靠地执行**。

> **Key Insight:** 模型是强大的引擎，但引擎本身不能决定去哪里、什么时候加速、什么时候刹车。Harness 就是那个决定"去哪"和"怎么去"的控制系统。

---

## 🔑 Key Terminology

| Term | Definition | Example |
|------|-----------|---------|
| **Agent Harness** | 包裹 AI 模型的管理层，负责工具、内存、安全和状态 | Salesforce Agentforce 的 Agent Harness 层 |
| **Long-Running Agent** | 需要在较长时间跨度（数小时/数天）内持续运行的 AI Agent | 监控 DevOps Pipeline 的 Agent |
| **Context Window** | 模型一次能处理的输入上限（以 token 计） | GPT-4o 的 128K，Claude 3.5 的 200K |
| **Function Hallucination** | 模型"幻觉"出一个不存在的工具函数并尝试调用 | Agent 说"我调用了 deleteRecord"但该函数不存在 |
| **API Timeout** | 外部 API 调用超时，Agent 陷入等待或重试循环 | 支付接口 30 秒无响应，Agent 卡死 |
| **Lifecycle Management** | Agent 的启动、暂停、恢复、终止管理 | Agent 任务完成后自动保存状态并退出 |
| **State Persistence** | 将 Agent 工作进度保存到外部存储，实现断点恢复 | Claude Code 的 claude-progress.txt |

---

## 💻 Code Examples

> ⚠️ 本课程重点在概念理解，不要求编程。代码示例仅用于帮助理解概念。

### Example 1: 模型 + Harness = 可运行的 Agent（概念图）

```
[用户输入]
     ↓
[Agent Harness 层]
 ├─ 上下文管理（Context Manager）
 ├─ 工具编排（Tool Orchestrator）
 ├─ 状态持久化（State Store）
 └─ 安全护栏（Safety Guardrails）
     ↓
[AI 模型（LLM）]
     ↓
[Harness 处理工具结果 → 重新决策]
     ↓
[外部世界：API / 文件系统 / 数据库]
```

### Example 2: 没有 Harness 的 Agent 会出现什么？（伪代码演示）

```python
# ❌ 没有 Harness 的情况：模型直接暴露给外部世界
def naive_agent(user_input, llm):
    context = user_input
    while True:
        response = llm.generate(context)
        
        # 模型可能输出任何内容——包括它"幻觉"出来的函数调用
        if "call_tool" in response:
            tool_name = extract_tool_name(response)  # 模型可能生成不存在的工具名！
            args = extract_args(response)
            result = execute_tool(tool_name, args)  # 这里会失败
            context += f"\n{result}"
        
        # 没有超时管理，没有状态保存，没有安全边界
        # 模型不知道何时该停，何时该重试

# ✅ 有 Harness 的情况：所有外部交互都经过 Harness
def harnessed_agent(user_input, llm, harness):
    context = harness.prepare_context(user_input)  # Harness 过滤上下文
    state = harness.load_state()                   # Harness 恢复上次进度
    
    while not harness.is_complete():
        response = llm.generate(context)
        
        if harness.should_call_tool(response):
            tool_calls = harness.parse_tool_calls(response)  # Harness 验证工具名
            for call in tool_calls:
                if not harness.validate_call(call):          # 安全检查
                    continue
                result = harness.execute_with_timeout(call) # 超时管理
                context = harness.update_context(result)    # 结果注入
                harness.save_state(state)                    # 定期保存
        
        if harness.is_safe_to_continue():
            continue
        else:
            harness.escalate_to_human()          # 高风险操作找人审批
```

**What's happening:** 左边的"没有 Harness"版本中，模型的每一次决策都直接被执行，包括它可能幻觉出来的函数。右边的版本中，Harness 作为中介层，**验证每一次工具调用**、**管理超时**、**保存状态**、**决定何时升级给人类**。

---

## ✏️ Hands-On Exercises

### Exercise 1: 识别约束（⏱️ ~10 min）

**Goal:** 列举你见过的 AI Agent 失败场景，对应到四大约束

**Instructions:**
1. 回忆你使用过的 AI Agent（客服机器人、代码助手、自动化流程等）
2. 列出它出问题的时刻——出了什么问题？
3. 尝试将每个问题对应到四大约束表（API 超时/内存耗尽/工具乱序/函数幻觉）

**Expected Output:**
```
场景：某自动化客服机器人
问题1：用户问了一个多步骤问题，机器人后来忘记之前说了什么
→ 对应：内存耗尽（Context Window 填满，早期信息丢失）

问题2：[你的答案]
→ 对应：[对应的约束]
```

---

### Exercise 2: 为野马设计 Harness（⏱️ ~15 min）

**Goal:** 用"野马与马具"的比喻，设计一个场景下的 Harness

**Instructions:**
1. 描述一个需要 AI Agent 的真实业务场景（例如：自动处理客户投诉）
2. 列出该场景下，AI 模型（野马）可能做出哪些"乱跑"的行为
3. 为每种"乱跑"行为设计对应的 Harness 管控措施

**Example Answer:**
```
场景：自动处理客户投诉邮件
"乱跑"行为：Agent 可能未经人工审批就发送退款邮件
Harness 管控：在发送任何涉及金钱的操作前，要求人类审批（Human-in-the-Loop）
```

---

### Exercise 3: 思考题（⏱️ ~5 min）

**Goal:** 深度思考 Harness 的必要性

**Question:** 一个完全可靠、从不犯错的 AI 模型，是否可以不需要 Harness？为什么？

> 提示：考虑（1）工具执行的安全性（2）外部世界的状态变化（3）成本控制

---

## 📖 Curated Resources

### Must-Read

1. **Agent Harness: The Infrastructure for Reliable AI** — Salesforce Agentforce
   - 🔗 https://www.salesforce.com/agentforce/ai-agents/agent-harness/ （本文课程来源）
   - Why: Salesforce 官方权威解读，定义了 Agent Harness 的行业术语

2. **What Is an Autonomous Agent?** — Salesforce
   - 🔗 https://www.salesforce.com/agentforce/ai-agents/autonomous-agents/
   - Why: 帮助你区分"聊天机器人"和"自主 Agent"的本质区别

---

## 🤔 Reflection Questions

1. **Comprehension:** 用一句话向一位非技术朋友解释：什么是 Agent Harness？为什么 AI 模型需要它？
   - *Think about：找一个日常比喻（汽车/员工/厨师……）*

2. **Application:** 你当前的工作或项目中，有没有可以用 Agent 自动化完成的任务？如果有，列出 3 个潜在的 Harness 设计需求。

3. **Critical Thinking:** 如果一个 AI 模型已经足够强大（推理能力极强），你认为 Harness 的哪些组件仍然不可或缺？为什么？

---

## ➡️ Next Steps

**Tomorrow: Day 2 — Agent vs Harness：核心区别与"法律体系"比喻**

明天我们将深入探讨 Agent（大脑）和 Harness（身体/环境）的精确边界，并用一个"法律体系"的比喻来理解它们的协作关系。

**Before moving on, make sure you can：**
- [ ] 说出 Agent Harness 的 3 个核心职责
- [ ] 解释为什么"模型足够好"不等于"产品足够好"
- [ ] 区分 Framework 和 Harness 的不同角色

[← 课程目录](../README.md) | [Next: Day 2: Agent vs Harness →](./day-02-agent-vs-harness.md)
