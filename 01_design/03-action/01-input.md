# ActionInput 设计文档

## 1. ActionInput 的定位

**ActionInput 是一次可执行行动的最小执行单元。**

它描述的不是“想做什么”，
而是**“现在要做什么，并且可以立刻做”**。

ActionInput **来源于 Cognition 的决策结果**，
但 **由 Runtime 负责生成、调度和交付**。

---

## 2. 角色分工澄清

在 ActionInput 的产生与使用过程中，各模块职责严格区分：

- **Cognition**

  - 形成决策、意图或行动方案
  - 不直接触发执行

- **Runtime**

  - 接收 Cognition 的输出
  - 判断执行时机与条件
  - 将决策转化为 ActionInput

- **Action**

  - 接收 ActionInput
  - 执行副作用
  - 返回执行事实

因此：

> **ActionInput 是 Runtime 认可并下发的执行指令，而非 Cognition 的原始输出。**

---

## 3. 设计原则

### 3.1 原子性

一次 ActionInput 对应一次**不可再分的执行尝试**。
不包含计划、不包含步骤列表。

### 3.2 去语义化

ActionInput 不携带：

- 行为动机（why）
- 目标描述（goal）
- 高层上下文解释

所有语义理解在 Cognition 阶段已经完成。

### 3.3 明确执行通道

ActionInput **必须明确指定执行方式**：

- 使用哪一个 Action 实现
- 使用哪一个执行器或 MCP
- 不允许 Action 再做选择

### 3.4 可调度性

ActionInput 必须是一个**可被 Runtime 操作的对象**：

- 可延迟
- 可丢弃
- 可重排
- 可记录

它不依赖 Cognition 的瞬时状态。

---

## 4. ActionInput 的本质定义（一句话）

> **ActionInput 是一次被 Runtime 批准、立即可执行的副作用请求。**

---

## 5. ActionInput 最小结构说明（Draft）

下面是一个**抽象层面的最小结构说明**，不绑定具体语言或实现。

### 5.1 核心结构

```
ActionInput {
  action_type
  target
  payload
  metadata
}
```

---

### 5.2 字段说明

#### action_type

- 描述要使用的 Action 实现类型
- 例如：command / mcp / hardware / ui
- 由 Runtime 根据 Cognition 输出确定

**作用：**
用于路由到正确的 Action 实现
Action 本身不进行类型推断

---

#### target

- 描述具体的执行目标或执行器
- 例如：

  - 本地命令执行器
  - 指定的 MCP Server
  - 特定设备或通道

**作用：**
明确副作用发生的位置
避免 Action 内部做任何“选择”

---

#### payload

- 执行所需的最小参数集合
- 与具体 Action 实现强相关
- Action 只读取、不解释、不补充

**作用：**
直接驱动副作用产生

---

#### metadata

- 由 Runtime 注入的执行相关信息
- 不参与执行逻辑本身

可能包含（但不限于）：

- action_id
- 调度时间戳
- 超时限制
- 执行优先级
- 追踪 / 审计信息

**作用：**
支持调度、审计与回溯
不影响 Action 的执行语义

---

## 6. 明确不包含的内容

ActionInput **不包含**：

- 决策依据
- 失败处理策略
- 重试规则
- 上下文理解
- 世界状态假设

这些内容若存在，说明职责边界已被破坏。

---

## 7. 与 ActionResult 的关系

ActionInput 只描述“要做什么”。
执行之后产生的所有结果，统一体现在 ActionResult 中。

ActionInput 与 ActionResult **不形成内部闭环**，
结果的意义必须交还 Cognition 处理。

---

## 8. 核心共识（一句话）

> **ActionInput 是 Runtime 对现实世界发出的执行指令，而不是大脑的想法。**
