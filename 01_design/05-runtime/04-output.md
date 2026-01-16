# Runtime · Output Model

> 本文档定义 Runtime 的**输出模型**。
> 本文档只描述 Runtime 对系统其他部分产生的**结构性影响**，
> 不描述流程、策略或实现细节。

---

## 1. 输出模型的设计原则

Runtime 的输出不是“返回值”，而是**裁决结果的生效**。

Runtime 不生成业务结果，
只对以下问题给出裁决性结论：

- 哪些行为在时间线上是合法的
- 当前时间是否可以推进
- 哪些跨时间状态被允许生效

---

## 2. Runtime 的输出类型

Runtime 的输出分为三类，且仅限这三类。

---

## 3. Action 裁决输出（Primary Output）

### 3.1 定义

Action 裁决输出表示 Runtime 对某个 Action 在当前时间线中的**生存裁决**。

### 3.2 裁决结果集合

Runtime 对任一 Action 只能输出以下结果之一：

- **Continue**：允许 Action 继续存在
- **Pause**：暂时停止 Action 的执行
- **Resume**：允许已暂停的 Action 继续执行
- **Terminate**：终止 Action，并使其退出时间线
- **Start**：允许新的 Action 实例进入时间线

### 3.3 结构约束

- Runtime 只裁决 Action 的合法性
- Runtime 不定义 Action 的内部执行方式
- Action 的所有生命周期变化必须源自 Runtime 的裁决输出

---

## 4. Epoch 推进输出（Temporal Output）

### 4.1 定义

Epoch 推进输出表示 Runtime 对时间结构的裁决结果。

### 4.2 输出内容

Runtime 在裁决完成后必须输出：

- 当前 Epoch 的**封存确认**
- 下一个 Epoch 的**创建与开启**

### 4.3 结构约束

- Epoch 推进是原子性的
- Epoch 不可回退、不可跳跃
- 所有 Action 裁决均发生在 Epoch 推进之前

---

## 5. 跨时间状态提交输出（State Commit Output）

### 5.1 定义

跨时间状态提交输出表示 Runtime 对跨 Epoch 状态变更的**生效许可**。

### 5.2 适用对象

- Memory 写入
- Control State 变更

### 5.3 结构约束

- Runtime 只“允许”状态生效
- Runtime 不生成状态内容
- 未被 Runtime 允许的状态变更不得跨 Epoch 生效

---

## 6. 非输出项（Explicit Non-Outputs）

为避免职责膨胀，Runtime 明确**不输出**以下内容：

- 认知结果本身
- 感知信息或其解释
- 行为策略或优先级
- 调度指令或资源分配结果

---

## 7. 输出模型的整体约束

通过上述输出模型，Runtime 保证：

- 所有行为变化均可追溯其裁决来源
- 时间推进具有明确且唯一的触发点
- 跨时间状态变化受统一裁决约束

---

> **Runtime 的输出不是“做了什么”，
> 而是“什么被允许在时间线上成立”。**
