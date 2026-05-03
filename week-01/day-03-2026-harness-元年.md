# Day 3: 2026 — 为什么 Harness 设计比选模型更重要

[← Previous: Day 2: Agent vs Harness](./day-02-agent-vs-harness.md) | [课程目录](../README.md) | [Next: Day 4: Long-Running Agents 三大失败原因 →](./day-04-long-running-agents-三大失败原因.md)

---

## 🎯 Learning Objectives

By the end of today, you will be able to：

1. **解释** 为什么 2025 年行业"卷模型"是错误的方向
2. **量化** Harness 质量对任务完成率的实际影响（40+ 百分点的差距）
3. **描述** 从"模型中心"到"基础设施中心"的范式转移
4. **理解** 为什么 2026 年是"Harness 元年"
5. **识别** 自己和团队在 Harness 设计上的投入是否足够

**Difficulty:** 🟡 Intermediate
**Estimated Time:** 30 minutes

---

## 📚 Core Concepts

### 3.1 一个让行业后悔的发现

2025 年初，整个 AI 行业形成了一个共识：**模型不够强，只要模型足够强，Agent 的一切问题都会迎刃而解。**

于是，数百亿美元的融资涌向模型训练，各团队争相比较模型 benchmark 分数，发布公告里满眼"超过 GPT-4"的标题。

然后到了 2026 年，行业发现了真相：

> **即使是最先进的模型，也无法克服 Agent scaffolding 的缺失。**
> （Even the most advanced model cannot overcome a lack of agent scaffolding.）

这个认知的代价是巨大的。很多团队花了 6-12 个月微调模型，最后发现瓶颈根本不在模型。一个公开的数据：

**LangChain 的真实案例** —— 改了 Harness，不改模型，benchmark 从 52.8% 提升到 66.5%，排名从第 30 名跃升到第 5 名。

这意味着：**两个团队用同一个模型，任务完成率可以差 40 个百分点。差距 100% 来自 Harness。**

### 3.2 范式转移：从 Model-Centric 到 Infrastructure-Centric

| 阶段 | 时间 | 行业共识 | 典型问题 |
|------|------|----------|----------|
| **模型中心时代** | 2024-2025 | "选更好的模型" | 模型越来越强，Agent 仍然失败 |
| **基础设施中心时代** | 2026+ | "Harness 是护城河" | 同样的模型，差距在 Harness |

为什么会这样？因为模型解决的是**推理质量问题**。但 Agent 在生产环境中的失败，**很少是推理质量问题**，更多是：

- 超时重试策略缺失
- 上下文窗口管理不当
- 工具选择错误（不是因为不会推理，而是因为上下文太乱）
- 状态在长时运行中丢失
- 权限边界没有强制执行

这些都不是模型的问题，是**执行环境**的问题。

### 3.3 可靠性的数学：为什么 20 步 Agent 成功率只有 36%

这是 Harness 重要性的最有力证据——一个简单的数学题：

**假设：** 每个步骤的成功率是 95%（已经是很高的数字）
**任务：** 需要 20 个有序步骤

**计算：** 0.95²⁰ ≈ 0.358

**结论：** 端到端任务完成率只有 **35.8%**

| 每步成功率 | 10 步后的成功率 | 20 步后的成功率 |
|-----------|-----------------|-----------------|
| 95% | 59.9% | 35.8% |
| 90% | 34.9% | 12.2% |
| 85% | 19.7% | **3.9%** |

**现实情况：** 大多数生产 Agent 任务超过 10 步，85% 的每步成功率是合理的估计。这意味着 **80%+ 的任务会在某个步骤失败**。

Harness 能做什么？它通过以下方式提升端到端成功率：
- **重试策略**（超时后指数退避重试）
- **断点保存**（失败后从上次保存点恢复，而不是从头开始）
- **中间验证**（每几步验证一次，不等最后才发现前面错了）
- **子任务分解**（把 20 步分解成 4 个 5 步的子任务，分别验证）

### 3.4 行业案例：Manus 和 LangChain 的"痛苦的重写"

**Manus** 是 2025 年初最火的通用自主 Agent 项目。它在社交媒体引发病毒式传播——一个 Agent 能帮你完成几乎任何数字任务。

但鲜为人知的是：**Manus 在 6 个月内重写了 5 次架构。**

原因？每一次新的前沿模型发布，Manus 团队都要重新设计周围的 Harness，因为新一代模型的最优 Harness 结构可能不同。

**LangChain** 的 Open Deep Research 也有类似的故事——4 次完整重构。

这说明两件事：
1. Harness 设计是一个需要持续迭代的专业工程领域
2. 如果你的 Harness 越做越复杂（而不是随模型改进趋于稳定），你可能过度设计了

> **Key Insight:** 2026 年的游戏规则是：**模型是商品（commodity），Harness 是护城河（moat）。** 不是选对模型就能赢，是做好 Harness 才能赢。

---

## 🔑 Key Terminology

| Term | Definition | Example |
|------|-----------|---------|
| **Model-Centric Design** | 以模型选择和训练为中心的系统设计哲学 | 2025 年前的行业主流思路 |
| **Infrastructure-Centric Design** | 以 Harness 和基础设施设计为中心的系统哲学 | 2026 年后的行业共识 |
| **Agent Scaffolding** | 支持 Agent 运行的底层基础设施总称 | 上下文管理、工具编排、安全护栏等 |
| **Compound Failure Rate** | 复合失败率：每步小概率错误累积成的大失败概率 | 0.85^10 = 20% 的端到端成功率 |
| **Benchmark Gap** | 相同模型不同 Harness 导致的性能差距 | LangChain 案例：40+ 百分点的差距 |
| **Harness Engineering** | 系统性地设计、测试和迭代 Agent Harness 的工程实践 | Manus 团队 6 个月 5 次重写的工作 |

---

## 💻 Code Examples

### Example 1: 复合失败率的计算

```python
# -*- coding: utf-8 -*-
"""
演示复合失败率：每步95%成功率，20步后只有35.8%的任务成功
"""

def calculate_end_to_end_success_rate(per_step_success_rate, num_steps):
    """
    计算端到端任务完成率
    
    Args:
        per_step_success_rate: 每步成功率（如 0.95 表示 95%）
        num_steps: 任务总步数
    Returns:
        端到端成功率（0-1 之间的小数）
    """
    return per_step_success_rate ** num_steps


# 不同场景下的端到端成功率
scenarios = [
    (0.95, 10, "95% 成功率，10 步"),
    (0.95, 20, "95% 成功率，20 步"),
    (0.90, 10, "90% 成功率，10 步"),
    (0.90, 20, "90% 成功率，20 步"),
    (0.85, 10, "85% 成功率，10 步（现实估计）"),
    (0.85, 20, "85% 成功率，20 步（恐怖真相）"),
]

print("=" * 55)
print(f"{'场景':<30} | {'成功率':>8} | {'失败率':>8}")
print("=" * 55)

for rate, steps, label in scenarios:
    success = calculate_end_to_end_success_rate(rate, steps)
    failure = 1 - success
    print(f"{label:<30} | {success:>7.1%} | {failure:>7.1%}")

print("=" * 55)
print()
print("💡 结论：20 步任务中，每步 85% 成功率意味着")
print("   只有 3.9% 的任务能端到端成功——接近 96% 会失败！")
print()

# 演示 Harness 如何改善：通过中间验证把长任务分段
print("=" * 55)
print("Harness 改善效果：每 5 步做一次验证")
print("=" * 55)

for rate, _, label in [(0.95, 10, "95% 成功率"), (0.85, 10, "85% 成功率")]:
    # 原始：10 步连续
    original = rate ** 10
    # 改善后：分成 2 个 5 步，中间验证恢复
    # 如果第 5 步失败，只重做后 5 步，不影响前 5 步
    # 每个 5 步段的成功率
    segment = rate ** 5
    # 两个段都成功的概率
    improved = segment * segment
    print(f"{label}: 连续10步 {original:.1%} → 分段验证 {improved:.1%} (提升 {(improved/original-1)*100:+.0f}%)")
```

**What's happening:** 这段代码演示了复合失败率的数学原理。第 3 个 print 语句说明了 Harness 的中间验证策略如何通过分段降低风险——如果第 5 步失败，只需要重做后 5 步，不需要从头开始。

### Example 2: 真实世界的模型对比 vs Harness 对比

```
团队 A（差 Harness）：      团队 B（好 Harness）：
模型：Claude 3.5          模型：Claude 3.5（相同）
Harness：简单             Harness：完善的验证+重试+状态持久化
─────────────────        ─────────────────
任务完成率：~35%          任务完成率：~75%
差距：40 个百分点！        （来源：LangChain 真实数据）

✅ 结论：完全相同的模型，差距全在 Harness
```

---

## ✏️ Hands-On Exercises

### Exercise 1: 计算你的 Agent 失败率（⏱️ ~10 min）

**Goal:** 量化你的 AI Agent 系统的可靠性现状

**Instructions:**
1. 估计你的 Agent 任务的平均步数（例：自动回复邮件可能需要 5 步）
2. 估计你的 Agent 每步成功率（例：工具调用成功率、API 可靠性）
3. 用公式计算你的端到端成功率

**公式：** `端到端成功率 = 每步成功率 ^ 总步数`

**Expected Output:**
```
我的 Agent 系统分析：

任务类型：自动生成销售报告
平均步数：8 步
每步成功率估计：90%（因为有外部 API 调用，有时会超时）

端到端成功率：90%^8 = 43%
失败率：57%

这意味着：每 10 个任务约有 6 个可能在某步失败。

改善方向：增加重试机制（当前没有）
```

---

### Exercise 2: 研究案例（⏱️ ~15 min）

**Goal:** 查找并分析一家公司的"Harness Engineering"实践案例

**Instructions:**
1. 搜索"Manus AI agent architecture"或"LangChain harness engineering"
2. 阅读该公司关于技术挑战和重写次数的公开陈述
3. 写出他们的核心 Harness 问题是什么？为什么重写了那么多次？

**提示：** 使用搜索引擎搜索相关关键词

---

### Exercise 3: 思维实验（⏱️ ~5 min）

**Question:** 如果你是一个 AI Agent 平台的 CTO，预算只够做一件事，你会：
- A. 买最新最强的模型（GPT-5 等）
- B. 花 6 个月把 Harness 从 60 分做到 90 分

请用今天学到的知识给出你的选择，并说明理由。

---

## 📖 Curated Resources

### Must-Read

1. **LangChain Terminal Benchmark Analysis** — LangChain Blog
   - 🔗 https://blog.langchain.dev/ （搜索 terminal benchmark 相关文章）
   - Why: 展示了纯改 Harness、不改模型带来的 benchmark 提升

2. **Manus 架构迭代历程** — 公开搜索
   - 🔗 通过网络搜索"Manus AI architecture rewrite"
   - Why: 了解前沿团队在 Harness 设计上付出的真实工程代价

---

## 🤔 Reflection Questions

1. **Comprehension:** 解释为什么"选更好的模型"无法解决 80% 的 Agent 生产失败问题。你需要至少提到两个具体原因。

2. **Application:** 根据 Day 3 的知识，评估你当前团队的资源分配：多少比例的工程时间花在了模型选择上，多少花在了 Harness 工程上？是否存在失衡？

3. **Critical Thinking:** "模型是商品，Harness 是护城河"这句话有没有局限性？你能想到它不适用的情况吗？

---

## ➡️ Next Steps

**Tomorrow: Day 4 — Long-Running Agents 三大失败原因**

今天学到了为什么 Harness 如此重要。明天我们将具体分析：Long-Running Agents（长时间运行的 Agent）在生产环境中为什么会失败？Context Drifting、Infinite Loops、Hallucinated Tool Usage——三大问题的机制和解决方案。

**Before moving on, make sure you can：**
- [ ] 解释 0.95^20 = 36% 的数学含义
- [ ] 说明 LangChain 如何在不换模型的情况下提升 14 个百分点
- [ ] 说出"模型是商品，Harness 是护城河"这句话的含义

[← Previous: Day 2: Agent vs Harness](./day-02-agent-vs-harness.md) | [课程目录](../README.md) | [Next: Day 4: Long-Running Agents 三大失败原因 →](./day-04-long-running-agents-三大失败原因.md)
