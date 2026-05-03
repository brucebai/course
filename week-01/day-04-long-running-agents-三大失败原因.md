# Day 4: Long-Running Agents 三大失败原因

[← Previous: Day 3: 2026：为什么 Harness 设计比选模型更重要](./day-03-2026-harness-元年.md) | [课程目录](../README.md) | [Next: Day 5: Context Rot 与 LLM Amnesia →](./day-05-context-rot-与-llm-amnesia.md)

---

## 🎯 Learning Objectives

By the end of today, you will be able to：

1. **识别** Long-Running Agents（长时运行 Agent）的典型应用场景
2. **解释** 三大失败原因：Context Drifting、Infinite Loops、Hallucinated Tool Usage
3. **理解** 每种失败的技术机制
4. **对应** 每种失败到具体的生产环境症状
5. **初步理解** Harness 如何缓解这些问题

**Difficulty:** 🟡 Intermediate
**Estimated Time:** 30 minutes

---

## 📚 Core Concepts

### 4.1 什么是 Long-Running Agent？

大多数 AI 对话（ChatGPT、Claude 对话）都是**短时交互**——用户发一条消息，Agent 回复一条，任务就结束了。这不需要 Harness。

但现代企业需要的，是**长时运行的 Agent**：

| 场景 | 运行时间 | 任务特点 |
|------|----------|----------|
| 销售外联 Agent | 几天~几周 | 管理整个外联流程，跟进潜在客户 |
| DevOps 监控 Agent | 持续运行 | 监控流水线，发现错误自动修复 |
| 数据ETL Agent | 数小时 | 跨多个数据源做数据清洗和迁移 |
| 研究 Agent | 数小时~数天 | 持续搜集信息，撰写报告 |

这些任务的共同特点：**不是在一次对话中完成的**。它们跨越多个会话、多天、甚至多周。同时，它们还涉及外部系统的状态变化——当 Agent 几个小时后回来时，外部世界可能已经变了。

正是这些特性，让 Long-Running Agents 成为 Harness 设计的主战场。

### 4.2 失败原因一：Context Drifting（上下文漂移）

**什么是 Context Drifting？**

随着 Agent 处理越来越多的信息，最重要的细节会逐渐"淹没"在之前步骤的噪声中。

**为什么会发生？**

AI 模型的上下文窗口（Context Window）是有限的。当 Agent 执行了 50 步之后，窗口里充满了：
- 每一步的工具输出
- 错误消息和重试日志
- 中间结果的中间状态
- 历史对话的残余

原本任务开始时用户提供的**原始目标**（"帮我优化这个慢查询"），在第 50 步时可能已经被推到了窗口的边缘，模型对它的注意力大大降低。

**症状表现：**
- Agent 在任务中途"忘记"了原始目标
- Agent 开始重复做某件事，因为不记得已经做过
- Agent 的输出质量在长任务后期明显下降
- Agent 在第 30 步做出的决策与第 1 步的策略矛盾

**类比：** 就像你在做一个长项目，开会时记了 100 页笔记，但当你需要找"项目最初的目标是什么"时，最初那两行关键信息早就被淹没在第 50 页的待办事项里了。

### 4.3 失败原因二：Infinite Loops（无限循环）

**什么是 Infinite Loops？**

Agent 不断重复同一个不成功的动作，因为它**没有"记忆"意识到自己已经尝试过这条路**。

**为什么会发生？**

典型的工具调用循环：

```
Agent 决定：调用 search_api(query="最佳咖啡机")
API 返回：抱歉，没有找到相关结果

Agent 决定：重试，换个关键词
Agent 调用：search_api(query="coffee machine")
API 返回：找到 3 个结果

Agent 决定：这些结果不够好，再搜一次
Agent 调用：search_api(query="best coffee maker 2024")
...
（Agent 陷入了不断搜索的循环，因为每次搜索都"还不够好"）
```

模型没有内置的循环检测机制。它不记得"我已经用同样的策略搜索了 5 次，每次结果都差不多"。

**症状表现：**
- Agent 重复相同的 API 调用
- Token 消耗快速飙升（每次调用都在消耗 Context Window）
- 任务似乎在"转圈"，没有任何进展
- API 调用次数远超合理范围

### 4.4 失败原因三：Hallucinated Tool Usage（工具幻觉）

**什么是 Hallucinated Tool Usage？**

模型尝试调用一个**不存在的函数**，或者传**错误的参数**，但表现得很自信。

**为什么会发生？**

LLM 的训练数据中有海量的代码，其中包含各种函数调用。但 tool-calling 的格式（JSON 描述工具）是**人工构造的**，在训练数据中稀疏存在。

结果是：模型对**真实存在的工具调用**（如 `search_web(query=...)`）和**它自己"编造"的工具调用**之间的区别，并没有我们想象的那么清晰。

**具体表现：**

1. **工具名幻觉：** Agent 说"我将调用 `get_user_email_by_id(id)`"，但这个函数根本不在工具列表中
2. **参数类型错误：** Agent 给 `send_email(to, subject, body)` 传了 `to="['a@b.com']`（列表作为字符串），而不是正确的格式
3. **API 路径幻觉：** Agent 调用 `GET /api/v2/users/payments` 但这个端点根本不存在

**最危险的是：** Agent 不会说"我不知道这个函数"，它会用一种非常自信的语气说"我已经成功调用了 xxx 函数"，但实际上什么都没发生——或者更糟，系统以为调用成功，但实际上参数是错的。

> **Key Insight:** Context Drifting 是"忘了"，Infinite Loops 是"重复"，Hallucinated Tool Usage 是"胡编"。三者加在一起——一个在长任务中逐渐遗忘目标、不断重复失败动作、还会编造不存在的函数调用的 Agent——就是没有 Harness 的真实结果。

---

## 🔑 Key Terminology

| Term | Definition | Example |
|------|-----------|---------|
| **Context Drifting** | 上下文漂移：关键信息随时间被无关信息淹没，导致目标丢失 | Agent 在第 50 步忘了第 1 步的目标 |
| **Infinite Loop** | 无限循环：Agent 重复相同失败动作，没有机制意识到循环存在 | 重复搜索 API 5 次，每次结果都"不够好" |
| **Hallucinated Tool Usage** | 工具幻觉：Agent 调用不存在的工具或传错误参数 | Agent 调用 `get_user_email()` 但该函数不存在 |
| **Context Window** | 上下文窗口：模型一次能处理的总 token 上限 | GPT-4o 的 128K，Claude 3.5 的 200K |
| **Long-Running Agent** | 需要在数小时到数天时间跨度内持续运行的自主 Agent | 销售外联 Agent、DevOps 监控 Agent |
| **Token Budget** | Token 预算：Context Window 中的有效信息容量 | 200K 窗口的实际可用空间 |

---

## 💻 Code Examples

### Example 1: Context Drifting 的可视化

```python
# -*- coding: utf-8 -*-
"""
演示 Context Window 中信息的"沉没"现象
"""

def simulate_context_window(scenario_name, total_steps, original_goal_token_count,
                           step_outputs, steps_per_pageview=2):
    """
    模拟 Context Window 的信息密度随任务进展的变化
    
    Args:
        total_steps: 总步数
        original_goal_token_count: 原始目标占用的 token 数
        step_outputs: 每步的工具输出大小（token）
        steps_per_pageview: 每次回顾时能看多少步（注意力窗口）
    """
    print(f"\n📊 场景：{scenario_name}")
    print(f"任务总步数：{total_steps}")
    print(f"Context Window 大小：200,000 tokens")
    print(f"原始目标大小：{original_goal_token_count} tokens")
    
    # 模拟上下文窗口内容
    window_content = []
    original_goal = f"[ORIGINAL_GOAL] 原始目标内容 ({original_goal_token_count} tokens)"
    window_content.append(original_goal)
    
    for step in range(1, total_steps + 1):
        output_size = step_outputs[step - 1]
        window_content.append(f"[STEP_{step}] 工具输出 ({output_size} tokens)")
        
        # 如果超出窗口限制，开始丢弃旧内容
        total_tokens = sum(
            original_goal_token_count if "[ORIGINAL_GOAL]" in c else
            step_outputs[i] if "[STEP_" in c else 100
            for i, c in enumerate(window_content)
        )
        
        if total_tokens > 200000:
            # 丢弃最早的非目标内容
            window_content.pop(1)  # 移除最早的步骤输出
    
    # 计算原始目标在当前窗口中的相对位置
    window_size = len(window_content)
    goal_position = list(enumerate(window_content)).index(
        (0, original_goal)
    ) if original_goal in window_content else -1
    
    print(f"\nContext Window 消耗：约 {sum(step_outputs[:total_steps]) + original_goal_token_count:,} tokens")
    print(f"窗口中的信息块数：{window_size}")
    
    if goal_position == -1:
        print(f"⚠️ 危险：原始目标已被挤出 Context Window！")
        print(f"   Agent 已经无法'看到'最初的任务目标。")
    elif goal_position < window_size // 2:
        print(f"⚠️ 警告：原始目标位置过于靠后（位置 {goal_position + 1}/{window_size}）")
        print(f"   模型对它的注意力显著降低。")
    else:
        print(f"✅ 安全：原始目标仍在 Context Window 前部。")
    
    return goal_position


# 运行模拟
print("=" * 60)
print("模拟：长时运行 Agent 的 Context Window 变化")
print("=" * 60)

# 场景1：10步任务，每步输出 5000 tokens
simulate_context_window(
    scenario_name="10步任务（正常）",
    total_steps=10,
    original_goal_token_count=500,
    step_outputs=[5000] * 10,
)

# 场景2：50步任务，每步输出 5000 tokens
simulate_context_window(
    scenario_name="50步任务（危险）",
    total_steps=50,
    original_goal_token_count=500,
    step_outputs=[5000] * 50,
)

# 场景3：100步任务
simulate_context_window(
    scenario_name="100步任务（严重）",
    total_steps=100,
    original_goal_token_count=500,
    step_outputs=[5000] * 100,
)
```

**What's happening:** 这段代码演示了 Context Window 的容量有限性。随着任务步数增加，原始目标被逐渐挤出窗口。注意 100 步任务中原始目标早已不在窗口内——这就是 Context Drifting 的技术原理。

---

## ✏️ Hands-On Exercises

### Exercise 1: 对号入座（⏱️ ~10 min）

**Goal:** 将生产环境中的问题现象对号入座到三大失败原因

**Instructions:** 阅读以下场景，判断它们分别属于哪种失败原因（Context Drifting / Infinite Loops / Hallucinated Tool Usage）

1. **场景 A：** 一个自动化客服 Agent 在处理长对话时，开始推荐与用户问题无关的产品
2. **场景 B：** Agent 反复调用同一个 API，说"上次没有返回正确结果"，但实际上已经调用了 8 次
3. **场景 C：** Agent 说"我将调用 `fetch_inventory_item(id)` 获取库存数据"，但系统中根本没有这个函数
4. **场景 D：** 用户说"我要取消订单"，Agent 在第 30 步时说"让我先查一下您的账户余额"，完全忘了取消订单这个请求

**Answers:**
- A：Context Drifting（长对话后期丢失原始意图）
- B：Infinite Loops（重复相同操作，无循环检测）
- C：Hallucinated Tool Usage（编造不存在的工具）
- D：Context Drifting（原始请求被淹没在历史记录中）

---

### Exercise 2: 设计缓解方案（⏱️ ~15 min）

**Goal:** 为每种失败原因设计 Harness 缓解措施

**Instructions:** 针对以下每种失败原因，列出你能想到的 Harness 解决方案

**Template:**
```
Context Drifting → 缓解方案：_______________
Infinite Loops → 缓解方案：_______________
Hallucinated Tool Usage → 缓解方案：_______________
```

**Hints（可选）：**
- Context Drifting 提示：定期把原始目标"刷新"到上下文窗口的前面
- Infinite Loops 提示：计数器 + 重复模式检测
- Hallucinated Tool Usage 提示：工具白名单 + 参数 schema 验证

---

### Exercise 3: 计算题（⏱️ ~5 min）

**Goal:** 用数学揭示 Infinite Loop 的危害

**Question:** 如果一个 Agent 每步的工具调用消耗 1000 tokens，每 5 步构成一次"循环尝试"，且没有循环检测。运行 50 步后：
- 消耗了多少 tokens？
- 其中有效工作的 tokens 是多少？
- 浪费在重复尝试的 tokens 是多少？

---

## 📖 Curated Resources

### Must-Read

1. **Long-Running Agents Fail Without a Harness** — Salesforce 原文章相关章节
   - 🔗 https://www.salesforce.com/agentforce/ai-agents/agent-harness/
   - Why: Salesforce 官方列出 Context Drifting / Infinite Loops / Hallucinated Tool Usage 三大原因的原文

2. **Context Window 的实际限制** — 相关 AI 论文
   - 🔗 可搜索 "LLM context window limitations long conversations"
   - Why: 学术角度理解为什么 LLM 的"记忆"本质上是有限的

---

## 🤔 Reflection Questions

1. **Comprehension:** 用你自己的话向一个非技术朋友解释：为什么"聪明的 AI"会在长时任务中"变笨"？

2. **Application:** 你见过的最糟糕的 AI Agent 生产失败案例，属于三大原因中的哪一个（或哪几个组合）？

3. **Critical Thinking:** 这三大失败原因中，你认为哪一个最危险、最难发现？为什么？

---

## ➡️ Next Steps

**Tomorrow: Day 5 — Context Rot 与 LLM Amnesia：被遗忘的上下文**

今天我们认识了三大失败原因。明天我们将深入其中最隐蔽的一类：**LLM Amnesia（健忘症）**——为什么模型"记不住"之前做过的事，以及 Context Rot（上下文腐烂）如何让 Agent 在长时任务中逐渐失控。

**Before moving on, make sure you can：**
- [ ] 准确说出三大失败原因的名称和表现
- [ ] 解释 Context Drifting 中"原始目标被淹没"的技术原理
- [ ] 说明为什么 Infinite Loops 在没有外部状态管理的情况下难以检测

[← Previous: Day 3: 2026：为什么 Harness 设计比选模型更重要](./day-03-2026-harness-元年.md) | [课程目录](../README.md) | [Next: Day 5: Context Rot 与 LLM Amnesia →](./day-05-context-rot-与-llm-amnesia.md)
