# Runtime · Workflow

> 本文档只描述 **Runtime 的结构事实与不变量**。
> 任何不能由结构直接推出的行为，均不属于 Runtime 的设计范围。

---

## 0. 设计前提（Structural Premise）

Runtime 是 Agent 的时间与因果裁决系统。

Runtime 不调度模块、不理解语义、不生成决策，
其唯一职责是：

> **在时间推进过程中，保证因果一致性。**

---

## 1. Epoch（因果时间单元）

Runtime 的基本工作单元是 **Epoch**。

### 1.1 Epoch 定义

- Epoch 表示 Runtime 所认定的一个“现在”
- Epoch 具有唯一标识
- 所有输入事件、裁决行为与状态提交 **必须归属于某一个 Epoch**

Epoch 的结束 **不由时间流逝决定**，
仅由因果条件触发。

---

## 2. Epoch 的结构状态

每一个 Epoch **有且仅有两种结构状态**：

```text
EpochState:
  - Open
  - Closed
```

这两种状态是 **结构态**，不是流程阶段。

---

## 3. Epoch · Open 状态

### 3.1 结构定义

Open 表示当前 Epoch 仍然有效。

### 3.2 不变量（Invariants）

在 Open 状态下，Runtime 必须满足：

- 所有新到达的输入事件 **归属于当前 Epoch**
- 不进行任何因果裁决
- 已存在的 Action 可继续执行（若其前提未被否定）

### 3.3 禁止事项

在 Open 状态下，Runtime **不得**：

- 主动结束 Epoch
- 基于输入内容做出语义判断
- 提交任何跨 Epoch 的状态变更

---

## 4. Epoch 从 Open 到 Closed 的转移

Epoch 只能从 Open 转移到 Closed。

该转移 **不可逆**，且 **必须原子完成**。

### 4.1 转移触发条件

只要满足以下任一条件，Epoch **必须立即关闭**：

- 出现可能进入现实因果链的 Cognition 结果
- 现有 Action 的前提被否定
- Runtime 的保守一致性策略触发

---

## 5. Epoch · Closed 状态

### 5.1 结构定义

Closed 表示当前 Epoch 已封口。

### 5.2 不变量（Invariants）

在 Closed 状态下，Runtime 必须满足：

- 当前 Epoch **不再接收任何输入事件**
- 所有新到达的输入事件 **自动归属于下一个 Epoch**
- 当前 Epoch 的输入集合固定不变

### 5.3 原子操作要求

Closed 状态 **必须在一次原子操作中完成以下步骤**：

1. 因果裁决

   - 裁决 Cognition 结果是否生效
   - 裁决 Action 是否继续存在

2. 状态提交

   - 提交允许生效的跨 Epoch 状态变更
   - 记录必要的历史信息

Closed **不得**：

- 等待外部事件
- 被观察为长期存在状态
- 被中断或重入

---

## 6. Epoch 推进规则

一旦 Closed 状态的原子操作完成：

- 当前 Epoch 被永久封存
- Runtime **必须立即创建下一个 Epoch**
- 新 Epoch 初始状态为 Open

不存在以下行为：

- 停留在 Closed
- 回退到前一 Epoch
- 跨越 Epoch 直接裁决

---

## 7. 输入归属规则（Structural Rule）

> **Runtime 在任一时刻，只允许 Open Epoch 接收输入。**

- Open Epoch：
  → 输入归属当前 Epoch

- Closed Epoch：
  → 输入归属下一个 Epoch

Runtime **不会丢弃输入**，
只改变其 Epoch 归属。

---

## 8. 并发与中断的结构约束

- **Perception**：持续并发，按 Epoch 归属
- **Cognition**：可并发执行，结果绑定 Epoch
- **Action**：可跨 Epoch 存在，必须可被 Runtime 中断

---

## 9. 结构性保证

通过上述结构约束，Runtime 保证：

- 世界始终只有一条时间线
- 裁决不会被未来事件污染
- 所有行为均可追溯其 Epoch 来源
- 系统在时间推进中保持因果一致性

---

## 10. 设计边界（Non-Goals）

Runtime 明确不承担：

- 语义理解
- 策略选择
- 优先级计算
- 行为规划

任何需要解释才能成立的逻辑，
都不应进入 Runtime。

---

> **Runtime 是时间结构，不是解释系统。**
