# Day 6: Harness 的六大核心组件概览

[← Previous: Day 5: Context Rot 与 LLM Amnesia](../day-05-context-rot-与-llm-amnesia.md) | [课程目录](../README.md) | [Next: Day 7: Demo → Production 的桥梁 →](./day-07-demo-to-production.md)

---

## 🎯 Learning Objectives

By the end of today, you will be able to：

1. **列举** Agent Harness 的六大核心组件
2. **解释** 每个组件的核心职责和与其他组件的关系
3. **理解** 六大组件如何协同防止 Day 4-5 中的三大失败原因
4. **评估** 一个现有 Agent 系统的组件健康度

**Difficulty:** 🟡 Intermediate
**Estimated Time:** 30 minutes

---

## 📚 Core Concepts

### 6.1 六大组件全景图

经过第一周的学习，我们已经知道 Harness 的重要性。现在，我们系统性地拆解它。

根据 Salesforce Agentforce、Harness Engineering 以及行业最佳实践，一个完整的 Agent Harness 由**六大核心组件**构成：

```
┌─────────────────────────────────────────────────┐
│              Agent Harness                       │
│  ┌──────────────────────────────────────────┐  │
│  │  FOUNDATION LAYER（基础层）              │  │
│  │  ① Context Engineering（上下文工程）     │  │
│  │  ② Tool Orchestration（工具编排）         │  │
│  │  ③ State Persistence（状态持久化）       │  │
│  └──────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────┐  │
│  │  SAFETY LAYER（安全层）                  │  │
│  │  ④ Evaluation（评估循环）               │  │
│  │  ⑤ Human-in-the-Loop（人机协同）        │  │
│  │  ⑥ Security & Governance（安全治理）     │  │
│  └──────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
         ↓                                          ↓
     [AI 模型] ← Agent（大脑）                  [外部世界]
```

### 6.2 基础层：让 Agent 能工作

**① Context Engineering（上下文工程）**

**职责：** 决定什么信息在每个时刻进入模型的上下文窗口，什么被压缩，什么被归档。

**核心问题：** Context Window 是有限的，但任务需求是无限的。

**关键子功能：**
- 上下文压缩（Compaction）：智能总结旧上下文
- 上下文刷新（Refresh）：定期把原始目标推回到窗口前部
- 工具输出 Offloading：大型输出存外部，窗口只保留摘要

**对应 Day 4-5 解决的问题：** Context Rot、LLM Amnesia

**② Tool Orchestration（工具编排）**

**职责：** 管理 Agent 可用的工具集，决定在每一步哪些工具可用，如何调用，调用结果如何处理。

**核心问题：** 工具太多 = 选择困惑 = 错误决策。工具太少 = 能力不足。

**关键子功能：**
- 动态工具作用域（Dynamic Tool Scoping）：根据任务阶段动态开启/关闭工具
- 参数验证（Argument Validation）：确保工具参数类型和格式正确
- 超时管理（Timeout Management）：防止工具调用永久挂起

**对应 Day 4-5 解决的问题：** Hallucinated Tool Usage（通过参数验证）、Infinite Loops（通过超时管理）

**③ State Persistence（状态持久化）**

**职责：** 在外部存储中保存 Agent 的工作进度，支持断点恢复。

**核心问题：** LLM 是无状态的，但 Agent 工作是有状态的。

**关键子功能：**
- 检查点保存（Checkpoint Saving）：定期保存进度
- 状态恢复（State Recovery）：崩溃后从检查点恢复
- 进度摘要（Progress Summarization）：用自然语言记录当前状态，供 Agent 恢复时阅读

**对应 Day 4-5 解决的问题：** LLM Amnesia

### 6.3 安全层：让 Agent 正确工作

**④ Evaluation（评估循环）**

**职责：** 在 Agent 执行过程中，持续验证其输出质量、进度正确性和目标一致性。

**核心问题：** Agent 不会自我评估自己的输出质量。GAN 的洞见：生成者和评估者必须是分开的。

**关键子功能：**
- 内部评估（Internal Eval）：规则驱动的自动化检查
- 外部评估（External Eval）：通过实际执行来验证（如使用 Playwright 打开页面检查）
- 硬性阈值（Hard Thresholds）：超过阈值的必须修复才能继续

**对应 Day 4-5 解决的问题：** Context Drifting（通过中间验证发现漂移）、Hallucinated Tool Usage（通过执行验证发现虚假调用）

**⑤ Human-in-the-Loop（人机协同）**

**职责：** 在高风险操作前暂停，等待人类审批；在 Agent 无法处理时升级给人类。

**核心问题：** Agent 犯错时可能代价很大，有些操作本质上是不可逆的。

**关键子功能：**
- 审批门（Approval Gates）：在特定操作类型前暂停
- 异常升级（Escalation）：Agent 无法决策时通知人类
- 实时监控（Live Monitoring）：人类可随时查看 Agent 正在做什么

**对应 Day 4-5 解决的问题：** All-or-Nothing Autonomy（审批门防止危险操作）

**⑥ Security & Governance（安全治理）**

**职责：** 定义 Agent 的权限边界、安全策略、合规要求，防止恶意使用和越权访问。

**核心问题：** 没有边界控制的 Agent 是潜在的安全风险。

**关键子功能：**
- 权限模型（Permission Model）：RBAC（基于角色的访问控制）
- 操作护栏（Guardrails）：防止执行危险操作（SQL 注入、Destructive commands）
- 审计日志（Audit Logs）：记录所有操作，供事后审计

---

## 🔑 Key Terminology

| Term | Definition | Example |
|------|-----------|---------|
| **Context Engineering** | 主动管理 LLM 上下文窗口内容的一门学科 | Claude Agent SDK 的自动压缩 |
| **Tool Orchestration** | Harness 中负责工具选择、调用、超时管理的组件 | Vercel 的 text-to-SQL Agent |
| **State Persistence** | 将 Agent 进度保存到外部存储以支持断点恢复 | Claude Code 的 claude-progress.txt |
| **Evaluation Loop** | Harness 中持续验证 Agent 输出的机制，分离 Generator 和 Evaluator | Anthropic 的 Playwright MCP QA |
| **Human-in-the-Loop** | 在高风险操作前引入人工审批的机制 | Replit 的 DROP DATABASE 审批门 |
| **Security & Governance** | Harness 中的权限、护栏和审计机制 | OAuth + API Key + 审计日志 |

---

## 🤔 Reflection Questions

1. **Comprehension:** 用六大组件逐一对应 Day 4 的三大失败原因：哪个组件解决哪个问题？

2. **Application:** 你当前使用的 Agent 系统，六大组件中哪几个是缺失的或不完整的？

3. **Critical Thinking:** 基础层（①②③）vs 安全层（④⑤⑥），哪个层更难做好？为什么？

---

## ➡️ Next Steps

**Tomorrow: Day 7 — 如何用 Harness 把"Demo Agent"变成"Production Agent"**

今天认识了六大组件的框架。明天我们将通过一个真实的转换案例，看看一个 Demo 级的 Agent 是如何在六大组件的加持下脱胎换骨的。

[← Previous: Day 5: Context Rot 与 LLM Amnesia](../day-05-context-rot-与-llm-amnesia.md) | [课程目录](../README.md) | [Next: Day 7: Demo → Production 的桥梁 →](./day-07-demo-to-production.md)
