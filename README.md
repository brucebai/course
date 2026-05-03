# Agent Harness 基础：Reliable AI 的基础设施
> 从 AI 模型到企业级 Agent 的完整基础设施指南

[![PWA Ready](https://img.shields.io/badge/PWA-Ready-c4825a)](https://bruce-bai.github.io/agent-harness-basics/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](./LICENSE)
[![Days](https://img.shields.io/badge/Course-10Days-blue.svg)](#course-outline)

## About This Course

你是否遇到过这样的情况：AI Agent 在演示中表现完美，但一上线生产就出现各种问题——API 超时、上下文丢失、工具调用乱序、甚至生成了不存在的函数？

这不是模型的问题。**这是 Harness（工具架/执行框架）缺失的问题。**

本课程基于 Salesforce Agentforce 团队发布的权威文章，系统讲解 Agent Harness 是什么、为什么 2026 年是"Harness 元年"、以及 Long-Running Agents 失败的三大根本原因。

**Duration:** 10 days (2 weeks)
**Daily commitment:** 30 minutes
**Language:** 简体中文（技术术语保留英文）
**Level:** 有一定基础（了解 AI Agent 基本概念）

## Learning Goals

学完之后，你将能够：

- ✅ 解释 Agent（大脑）和 Harness（身体/环境）的核心区别
- ✅ 理解为什么 2026 年 Harness 设计比选择模型更重要
- ✅ 识别 Long-Running Agents 三大失败原因：Context Drifting、Infinite Loops、Hallucinated Tool Usage
- ✅ 说明 Context Rot 和 LLM Amnesia 的机制
- ✅ 掌握 Agent Harness 的六大核心组件及其相互作用

## Course Outline

| 周次 | 天数 | 主题 | 类型 |
|------|------|------|------|
| Week 1 | Day 1 | 什么是 Agent Harness？— 从 AI 模型到企业级 Agent | Foundations |
| Week 1 | Day 2 | Agent vs Harness — 核心区别与"法律体系"比喻 | Foundations |
| Week 1 | Day 3 | 2026：为什么 Harness 设计比选模型更重要 | Foundations |
| Week 1 | Day 4 | Long-Running Agents 三大失败原因 | Foundations |
| Week 1 | Day 5 | Context Rot 与 LLM Amnesia — 被遗忘的上下文 | Foundations |
| Week 2 | Day 6 | Harness 的六大核心组件概览 | Building |
| Week 2 | Day 7 | 如何用 Harness 把"Demo Agent"变成"Production Agent" | Building |
| Week 2 | Day 8 | 工具调用验证与安全防护 | Building |
| Week 2 | Day 9 | 状态持久化 — 让 Agent 记住自己的进度 | Building |
| Week 2 | Day 10 | Capstone — 设计你的第一个 Agent Harness 架构 | Project |

## 🚀 Get Started

### Learn Online

访问 PWA 课程网站（可安装，离线可用）：

```
https://bruce-bai.github.io/agent-harness-basics/
```

### Learn Locally

```bash
# Clone the course repo
git clone https://github.com/bruce-bai/agent-harness-basics.git
cd agent-harness-basics

# Open daily study guides
open week-01/day-01-什么是-agent-harness.md

# Or open the course website
open website/index.html
```

## 📦 Repo Structure

```
agent-harness-basics/
├── README.md
├── .github/workflows/deploy.yml
├── week-01/
│   ├── day-01-什么是-agent-harness.md
│   ├── day-02-agent-vs-harness.md
│   ├── day-03-2026-harness-元年.md
│   ├── day-04-long-running-agents-三大失败原因.md
│   └── day-05-context-rot-与-llm-amnesia.md
├── week-02/
│   ├── day-06-六大核心组件概览.md
│   ├── day-07-demo-to-production.md
│   ├── day-08-工具调用验证与安全防护.md
│   ├── day-09-状态持久化.md
│   └── day-10-capstone.md
├── flashcards/
│   └── flashcards.json
├── diagrams/
└── website/
    ├── index.html
    ├── styles.css
    ├── main.js
    ├── manifest.json
    ├── sw.js
    ├── offline.html
    ├── data/
    ├── modules/
    └── icons/
```

## Resources

- [Agent Harness: The Infrastructure for Reliable AI](https://www.salesforce.com/agentforce/ai-agents/agent-harness/) — Salesforce Agentforce 官方文章
- [What Is an AI Agent?](https://www.salesforce.com/agentforce/ai-agents/autonomous-agents/) — Autonomous Agents 概念解析

## License

MIT — See [LICENSE](./LICENSE).
