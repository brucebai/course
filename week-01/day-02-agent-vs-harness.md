# Day 2: Agent vs Harness — 核心区别与"法律体系"比喻

[← Previous: Day 1: 什么是 Agent Harness](./day-01-什么是-agent-harness.md) | [课程目录](../README.md) | [Next: Day 3: 2026：为什么 Harness 设计比选模型更重要 →](./day-03-2026-harness-元年.md)

---

## 🎯 Learning Objectives

By the end of today, you will be able to：

1. **精确区分** Agent（负责 what/why）和 Harness（负责 how/where）的职责边界
2. **描述** Agent 和 Harness 在"概率性 vs 确定性"维度上的本质差异
3. **运用**"法律体系"比喻来向非技术背景的人解释两者关系
4. **理解** 为什么这种区分对系统设计至关重要

**Difficulty:** 🟢 Beginner
**Estimated Time:** 30 minutes

---

## 📚 Core Concepts

### 2.1 "法律体系"的比喻

理解 Agent 和 Harness 的关系，最好的比喻是**法律体系**：

想象 AI 模型是一个经验丰富的**律师**。这个律师：
- 📚 **知识渊博**：熟读法律条文，有深厚的法学功底
- 🧠 **擅长推理**：能够运用逻辑分析复杂案件
- 💬 **表达流畅**：能用清晰的语言阐述法律观点

但一个律师，**仅凭自己**，无法让法律系统运转。你还需要：

| 组成部分 | 对应角色 | 职责 |
|----------|----------|------|
| 法律体系 | **法院和法官** | 提供结构，让律师的工作有章可循 |
| 法律条文 | **Harness 的规则集** | 规定什么能做、什么不能做 |
| 陪审团 | **评估/验证机制** | 帮助判断律师的论证是否公正 |
| 律师 | **Agent（AI 模型）** | 提供知识和推理，但受体系约束 |

**Agent Harness = 司法体系**。它不是替代律师，而是确保律师在**法律框架内**工作、**公正地**论证、**正确地**引用法律。

### 2.2 Agent 与 Harness 的精确边界

这不是一个学术区分，而是有实际工程意义的划分：

| 维度 | Agent（大脑/The Brain） | Harness（身体/环境/The Body） |
|------|--------------------------|-------------------------------|
| **核心职责** | **推理**：决定下一步做什么、为什么做 | **执行**：管理工具调用、状态、外部连接 |
| **行为模式** | 概率性（Probabilistic）：基于概率预测最佳行动 | 确定性（Deterministic）：遵循硬编码规则、安全检查 |
| **"说什么"** | "我要解决 X 问题，策略是 A → B → C" | "A 步骤成功，现在执行 B，但 C 需要人工审批" |
| **"怎么控制"** | 模型权重 / System Prompt | 代码逻辑 / 安全规则 / 超时配置 |
| **类比** | 赛车手（决定路线、速度） | 赛车（引擎管理、安全系统、刹车片） |

### 2.3 Harness 负责"How"和"Where"

Salesforce 的定义说得很清楚：

> **Agent 负责"what"和"why"，Harness 负责"how"和"where"。**

举例：给 Agent 一个任务"帮我优化这个数据库查询"。

**Agent 决定：**
- What：要优化查询语句
- Why：因为当前查询扫描了全表，性能差

**Harness 决定：**
- How：Agent 是否可以直接执行 SQL？需要只读权限？执行前要不要备份？
- Where：查询在哪个数据库实例上执行？生产库还是测试库？

这个"How"和"Where"的分离，是系统**可预测性**和**安全性**的来源。

### 2.4 两种风格的 AI 系统

为了加深理解，看一个具体的例子——**Anthropic 的 Claude**：

同一个 Claude 模型，运行在三个完全不同的产品中：

| 产品 | 模型 | Harness 差异 | 用户感知 |
|------|------|-------------|----------|
| Claude.ai 对话 | Claude 3.5 Sonnet | 对话历史管理、安全过滤器、内容策略 | 聊天界面 |
| Claude Code | Claude 3.6 Sonnet | 文件系统访问、Terminal 会话、多会话状态 | 终端编程 Agent |
| Enterprise API | Claude 3.5 API | 自定义工具集、业务规则、审批流 | 企业级集成 |

**相同的模型权重，完全不同的行为**。不同的，就是 Harness。

> **Key Insight:** 如果你换了一个"更聪明的模型"但没有改进 Harness，你的 Agent 体验不一定提升。但如果你改进了 Harness，即使不换模型，体验也可能大幅提升。2026 年行业学到的最重要的教训：**模型是商品，Harness 是护城河。**

---

## 🔑 Key Terminology

| Term | Definition | Example |
|------|-----------|---------|
| **Agent（The Brain）** | 负责推理和决策的 AI 模型组件，回答"做什么"和"为什么做" | 调用 LLM 生成下一步计划 |
| **Harness（The Body）** | 负责执行环境和控制的软件层，回答"怎么做"和"在哪里做" | 验证工具调用、管理超时、保存状态 |
| **Probabilistic（概率性）** | 基于概率预测，不保证每次结果相同 | LLM 每次可能选择不同的工具 |
| **Deterministic（确定性）** | 遵循硬编码规则，每次输入产生可预测的输出 | Harness 的安全检查逻辑 |
| **What & Why** | Agent 的决策范围——目标确定和策略选择 | Agent 决定"先搜索还是先计算" |
| **How & Where** | Harness 的执行范围——执行方式和执行位置 | Harness 决定"在哪个数据库实例执行" |
| **Harness Engineering** | 构建和维护 Agent Harness 的专业工程学科 | Vercel、Manus、LangChain 团队的核心工作 |

---

## 💻 Code Examples

### Example 1: Agent 和 Harness 的决策边界

```python
# 这段代码演示了 Agent 和 Harness 的决策分离

# === AGENT（大脑）：决定"做什么"和"为什么做" ===
def agent_decide(task, context):
    """
    Agent 的推理逻辑：
    输入：任务 + 上下文
    输出：决策（计划、工具选择理由）
    """
    # 模型思考：基于当前上下文，我应该怎么解决这个任务？
    plan = llm.generate(
        system_prompt="你是一个数据库优化专家。分析用户查询，给出优化建议。",
        user_prompt=f"任务：{task}\n当前上下文：{context}"
    )
    return plan
    # Agent 不知道也不关心：这个查询会在哪个环境执行，
    # 有没有权限，是否需要备份——那是 Harness 的职责


# === HARNESS（身体）：决定"怎么做"和"在哪里做" ===
class DatabaseHarness:
    def should_execute(self, agent_decision, tool_name):
        """ Harness 做安全检查：这是合法操作吗？ """
        # 高风险操作需要审批
        if tool_name in ["DROP_TABLE", "DELETE_RECORD"]:
            return self.request_human_approval(agent_decision)
        
        # 检查工具是否在白名单内
        if tool_name not in self.allowed_tools:
            return False
        
        # 检查执行位置（生产库 vs 测试库）
        if agent_decision.target == "production_db":
            return self.request_production_token(agent_decision)
        
        return True
    
    def execute_with_guardrails(self, tool_call):
        """ Harness 执行：超时管理 + 错误处理 + 状态保存 """
        try:
            result = self.run_with_timeout(tool_call, timeout=30)
            self.save_checkpoint()  # 定期保存进度
            return result
        except TimeoutError:
            self.retry_with_backoff(tool_call, max_retries=3)
        except PermissionError:
            self.escalate("权限不足，需要人工介入")
```

**What's happening:** Agent 专注推理，生成策略和建议。Harness 负责将策略转化为安全的动作——检查权限、管理超时、保存状态。两者的边界是清晰的。

### Example 2: 对比"聊天机器人"和"自主 Agent"的控制流

```
传统聊天机器人：
用户 → [直接发送给 LLM] → LLM 回复 → 用户
             ↑
         没有 Harness

自主 Agent（有 Harness）：
用户 → [Harness 接收] → [Agent 推理] → [Harness 验证] → [执行工具]
                                                  ↓
                                            [返回结果给 Agent]
                                                  ↓
                                            [Agent 决定下一步]
← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ←
         [Harness 持续监控执行上下文和状态]
```

---

## ✏️ Hands-On Exercises

### Exercise 1: 角色扮演（⏱️ ~10 min）

**Goal:** 用"法律体系"比喻分析一个 AI Agent 系统的组成

**Scenario:** 一个自动股票交易 Agent

**Instructions:**
1. 列出该 Agent 需要做的决策（哪些是 what/why，哪些是 how/where）
2. 指出 Harness 需要负责哪些控制点

**Expected Output:**
```
自动股票交易 Agent

Agent（律师）的决策：
- what/why：用户要求"当 Tesla 股价跌破 $200 时自动卖出"
- Agent 决定：策略是"当 price < 200 时执行 sell"

Harness（司法体系）的控制：
- How：是否允许全仓卖出？还是只能卖 50%？
- Where：在哪个券商账户执行？
- 安全护栏：每天最大交易次数限制
- 断点恢复：如果系统崩溃，已提交的交易能否撤销？
```

---

### Exercise 2: 分类练习（⏱️ ~10 min）

**Goal:** 区分以下决策属于 Agent 还是 Harness

**Instructions:** 判断每个决策属于 Agent（what/why）还是 Harness（how/where），并说明理由：

1. "我应该先搜索相关文档还是直接执行 SQL？"
2. "这个 SQL 只能对测试库执行，不能碰生产库"
3. "用户的问题是要查询销售报表数据"
4. "超过 10000 元的退款需要主管审批"
5. "我决定用 3 个步骤解决这个问题"
6. "API 超时重试使用指数退避策略"

**Answers:**
1. Agent — 推理策略选择（what/why）
2. Harness — 执行位置约束（how/where）
3. Agent — 理解任务目标（what/why）
4. Harness — 权限边界和安全规则（how/where）
5. Agent — 任务分解决策（what/why）
6. Harness — 错误处理规则（how/where）

---

### Exercise 3: 思考题（⏱️ ~10 min）

**Question:** 有人说"Harness 是 AI 系统的操作系统（OS）"。你同意吗？类比操作系统（进程管理、内存管理、文件系统、系统调用），分别说明 Harness 的哪些功能可以类比为操作系统的哪些组件。

> 提示：考虑进程调度（Process Scheduling）、内存管理（Memory Management）、文件系统（File System）、系统调用（System Calls）等维度

---

## 📖 Curated Resources

### Must-Read

1. **Agent vs Agent Harnesses: Key Differences** — Salesforce 官方表格
   - 🔗 https://www.salesforce.com/agentforce/ai-agents/agent-harness/ （见原文对比表格）
   - Why: 官方权威对比，含 Agent（大脑）和 Harness（身体）的详细对照表

2. **What Is Agentic AI?** — Salesforce
   - 🔗 https://www.salesforce.com/agentforce/what-is-agentic-ai/
   - Why: 区分"普通聊天 AI"和"Agentic AI"的概念演进

---

## 🤔 Reflection Questions

1. **Comprehension:** 用 Phil Schmid 的比喻（Model = CPU，Harness = OS）解释：为什么 CPU 加上操作系统才能变成一台可用的电脑，而不是发热元件？

2. **Application:** 你现在是否有一个正在开发或使用的 AI 系统？它的 Agent 部分和 Harness 部分分别是哪些代码/模块？Harness 部分是否足够健壮？

3. **Critical Thinking:** 如果 Agent 的推理能力接近完美（100% 准确），Harness 中的哪些功能仍然不可或缺？列出至少 3 个。

---

## ➡️ Next Steps

**Tomorrow: Day 3 — 2026：为什么 Harness 设计比选模型更重要**

我们将揭示一个让很多团队后悔的真相：2025 年大家都在卷模型，2026 年发现真正的瓶颈在 Harness。数据会告诉你：为什么两个团队用同一个模型，任务完成率可以差 40 个百分点。

**Before moving on, make sure you can：**
- [ ] 准确说出 Agent 负责 what/why，Harness 负责 how/where
- [ ] 用"法律体系"或"CPU+OS"的比喻解释两者关系
- [ ] 说出至少 2 个 Harness 比 Agent 更"确定性"的例子

[← Previous: Day 1: 什么是 Agent Harness](./day-01-什么是-agent-harness.md) | [课程目录](../README.md) | [Next: Day 3: 2026：为什么 Harness 设计比选模型更重要 →](./day-03-2026-harness-元年.md)
