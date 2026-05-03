# Day 8: 工具调用验证与安全防护

[← Previous: Day 7: Demo → Production 的桥梁](./day-07-demo-to-production.md) | [课程目录](../README.md) | [Next: Day 9: 状态持久化 →](./day-09-状态持久化.md)

---

## 🎯 Learning Objectives

By the end of today, you will be able to：

1. **解释** 工具调用的三层验证机制
2. **识别** Hallucinated Tool Usage 的典型模式
3. **设计** 针对高危操作的审批门（Approval Gates）
4. **理解** 参数验证的实践方法

**Difficulty:** 🟡 Intermediate
**Estimated Time:** 30 minutes

---

## 📚 Core Concepts

### 8.1 为什么工具调用需要验证？

在 Day 4 中我们认识了 Hallucinated Tool Usage——Agent 调用不存在的工具或传错误参数。但"识别问题"不等于"解决问题"。

工具调用的验证分为三层：

**第一层：白名单验证（Which）**
- 这个工具名称是否在 Harness 允许列表中？
- 如果不在，直接拒绝

**第二层：参数验证（How）**
- 参数类型是否正确？（string vs number vs array）
- 必填参数是否都已提供？
- 参数值是否在合法范围内？

**第三层：执行环境验证（Where）**
- 这个工具是否允许在当前上下文中执行？
- （例如：只读环境不允许 write 操作）

### 8.2 参数验证的具体方法

```python
# 一个简单的参数校验器
class ParameterValidator:
    def validate(self, tool_name, args, schema):
        errors = []
        
        for param_name, param_schema in schema.items():
            # 必填参数检查
            if param_schema.get("required") and param_name not in args:
                errors.append(f"Missing required parameter: {param_name}")
                continue
            
            if param_name in args:
                value = args[param_name]
                expected_type = param_schema.get("type")
                
                # 类型检查
                if expected_type == "string" and not isinstance(value, str):
                    errors.append(f"{param_name} must be string, got {type(value)}")
                elif expected_type == "number" and not isinstance(value, (int, float)):
                    errors.append(f"{param_name} must be number, got {type(value)}")
                elif expected_type == "array" and not isinstance(value, list):
                    errors.append(f"{param_name} must be array, got {type(value)}")
                
                # 范围检查
                if "min" in param_schema and value < param_schema["min"]:
                    errors.append(f"{param_name} must be >= {param_schema['min']}")
                if "max" in param_schema and value > param_schema["max"]:
                    errors.append(f"{param_name} must be <= {param_schema['max']}")
        
        return errors
```

### 8.3 审批门：防止 All-or-Nothing Autonomy

Day 4 中提到 Replit 的案例：Agent 执行了 `DROP DATABASE` 生产操作，因为没有任何权限边界。

**审批门（Approval Gate）** 的设计原则：

| 高风险操作 | 审批策略 |
|-----------|----------|
| DELETE / DROP 操作 | 必须人工审批，且提供操作预览 |
| 外部网络请求 | 可自动执行，但记录审计日志 |
| 数据导出 | 需要部门主管审批 |
| 财务操作（退款、转账） | 金额超过阈值必须审批 |

> **Key Insight:** 审批门不是"阻碍"Agent 工作，而是"确保"Agent 的高风险决策是经过人类确认的。一个好的审批门设计 = 让低风险操作快速通过 + 让高风险操作停下来等确认。

---

## 🔑 Key Terminology

| Term | Definition | Example |
|------|-----------|---------|
| **Whitelist Validation** | 白名单验证：只允许列表中的工具被调用 | 16个专业工具 vs 4个通用工具 |
| **Parameter Schema** | 参数模式：定义每个工具的参数类型、是否必填、取值范围 | `{"type": "string", "required": true, "maxLength": 100}` |
| **Approval Gate** | 审批门：在高风险操作前暂停等待人工确认的机制 | DROP DATABASE 前必须点击"确认删除" |
| **Guardrail** | 护栏：防止 Agent 执行危险操作的规则 | 禁止 Agent 直接执行 DELETE without WHERE |

---

## 🤔 Reflection Questions

1. **Comprehension:** 解释为什么参数验证比工具名验证更重要？

2. **Application:** 你见过的最危险的 Agent 越权操作是什么？如果你是那个系统的 Harness 工程师，你会怎么加护栏？

---

## ➡️ Next Steps

**Tomorrow: Day 9 — 状态持久化**

[← Previous: Day 7: Demo → Production](./day-07-demo-to-production.md) | [课程目录](../README.md) | [Next: Day 9: 状态持久化 →](./day-09-状态持久化.md)
