# Day 7: Demo → Production — 如何用 Harness 把"Demo Agent"变成"Production Agent"

[← Previous: Day 6: Harness 的六大核心组件概览](./day-06-六大核心组件概览.md) | [课程目录](../README.md) | [Next: Day 8: 工具调用验证与安全防护 →](./day-08-工具调用验证与安全防护.md)

---

## 🎯 Learning Objectives

By the end of today, you will be able to：

1. **区分** Demo Agent 和 Production Agent 的核心差异
2. **应用** Vercel 的"工具哲学"——少即是多
3. **设计** 一个从 Demo 到 Production 的 Harness 改进路线图
4. **理解** 为什么 Demo Agent 和 Production Agent 的工具集策略截然不同

**Difficulty:** 🟡 Intermediate
**Estimated Time:** 30 minutes

---

## 📚 Core Concepts

### 7.1 Demo Agent 的甜蜜陷阱

大多数 Agent 项目都是从 Demo 开始的。Demo 很美好：

- 数据是干净的（精心挑选的样例数据）
- 场景是简单的（只演示核心功能）
- 环境是受控的（没有意外的网络状况）
- 用户是友好的（知道怎么配合 Agent）

但生产环境呢？

| 维度 | Demo 环境 | Production 环境 |
|------|-----------|----------------|
| 数据 | 干净、格式统一 | 脏数据、缺失字段、格式混乱 |
| 场景 | 核心流程 | 边缘 case、异常流程 |
| 网络 | 稳定 | API 超时、断连、重试 |
| 时长 | 几分钟 | 数小时到数天 |
| 用户意图 | 明确 | 模糊、随时改变 |

**结论：Demo 通过 ≠ Production 通过。**

### 7.2 Vercel 的悖论：减少 80% 工具，获得 100% 成功率

Vercel 的 v0 团队曾开发一个 text-to-SQL Agent。最初，他们设计了 16 个专业工具：

- schema_lookup（查询数据库 Schema）
- query_validator（验证 SQL 语法）
- error_recovery（错误恢复例程）
- ...等

结果是：Agent 脆弱、缓慢、维护成本高。

他们的解决方案出乎意料：**删掉 80% 的工具，只保留一个——bash 命令执行**。

结果呢？

| 指标 | 之前（16 工具） | 之后（4 工具） | 改善 |
|------|----------------|----------------|------|
| 成功率 | 80% | 100% | +20% |
| 响应时间 | 274 秒 | 77 秒 | 3.5x 更快 |
| Token 消耗 | ~102K | ~61K | -37% |
| 执行步数 | ~12 | ~7 | -42% |

**Lesson：** "不要与重力对抗（Don't fight gravity）"——文件系统是一个强大的、通用的抽象。在它之上构建专门的工具，往往是在重复发明轮子，同时增加了维护负担和模型的选择困惑。

### 7.3 从 Demo 到 Production 的 Harness 检查清单

当你准备将 Demo Agent 投入生产时，用这份检查清单评估你的 Harness：

**① Context Engineering 检查：**
- [ ] 原始指令是否能在 50 步后仍然清晰？
- [ ] 是否有上下文压缩机制？
- [ ] 大型工具输出是否被 offload 而非注入上下文？

**② Tool Orchestration 检查：**
- [ ] 工具总数是否超过 10 个？（考虑精简）
- [ ] 工具参数是否有 schema 验证？
- [ ] 是否有超时和重试策略？

**③ State Persistence 检查：**
- [ ] Agent 是否会定期保存检查点？
- [ ] 崩溃后能否从断点恢复？
- [ ] 进度是否以结构化方式存储（而非纯文本）？

**④ Evaluation 检查：**
- [ ] 是否有中间验证步骤（而非等最后一起检查）？
- [ ] Generator 和 Evaluator 是否分离？

**⑤ Human-in-the-Loop 检查：**
- [ ] 危险操作前是否有审批门？
- [ ] Agent 无法决策时是否有升级路径？

**⑥ Security 检查：**
- [ ] 是否有权限边界（Agent 不能做什么）？
- [ ] 是否有审计日志？

> **Key Insight:** Demo Agent 的失败几乎总是在 Harness，不是在模型。把 Demo 转 Production 的过程，本质上是一个 Harness 健壮化过程。

---

## 🔑 Key Terminology

| Term | Definition | Example |
|------|-----------|---------|
| **Demo Trap** | Demo 环境与 Production 环境的巨大差异导致的"Demo 通过但生产失败"现象 | Vercel 16工具Agent在Demo成功但Production失败 |
| **Don't Fight Gravity** | Vercel 的工具哲学：利用已有的强大抽象（filesystem）而非重新发明轮子 | 用 bash + filesystem 替代 16 个专门工具 |
| **Harness Hardening** | 将 Demo 级的 Harness 加强为 Production 级的过程 | 添加重试、超时、状态持久化、审批门 |

---

## ✏️ Hands-On Exercises

### Exercise 1: Vercel 案例分析（⏱️ ~10 min）

**Goal:** 深入分析 Vercel 为什么减少工具后效果更好

**Question:** 用你自己的话解释：为什么更多的工具反而让 Agent 表现更差？

### Exercise 2: 检查你的 Agent（⏱️ ~15 min）

**Goal:** 用检查清单评估你当前或一个示例 Agent 系统

**Scenario:** 一个自动化客服 Agent，需要回答用户关于订单状态、退款政策、产品规格的问题。

使用上面的 6 维度检查清单，评估它是否已准备好进入生产。

---

## ➡️ Next Steps

**Tomorrow: Day 8 — 工具调用验证与安全防护**

今天学到了 Demo 到 Production 的转换思路。明天我们将深入工具编排层：如何在实践中断言工具调用、验证参数、防止 Hallucinated Tool Usage。

[← Previous: Day 6: Harness 的六大核心组件概览](./day-06-六大核心组件概览.md) | [课程目录](../README.md) | [Next: Day 8: 工具调用验证与安全防护 →](./day-08-工具调用验证与安全防护.md)
