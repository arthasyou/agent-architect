# Runtime · Perception 输出处理

## 1. 文档目的

本文档用于**补齐 Perception 的系统闭环**，明确以下问题：

- Perception 的输出由谁接收
- 输出在系统中如何被管理
- 何时、以何种方式触发 Cognition

本文档仅描述**设计思想与职责边界**，不涉及具体数据结构、调度算法或实现细节。

---

## 2. Runtime 在整体架构中的定位

Runtime 是 Agent 的**时间、注意力与调度系统**。

在 Perception、Cognition、Action 之间，Runtime 承担的是**中枢协调角色**，而不是业务处理角色。

在感知—认知链路中，Runtime 是：

> **Perception 输出的唯一消费者与调度者。**

Perception 不直接与 Cognition 或 Action 交互，其所有输出必须先进入 Runtime。

---

## 3. 为什么 Perception 的闭环必须在 Runtime 中完成

Perception 的职责是产生统一、低语义的内部信号。

但 Perception 本身：

- 无法感知系统整体状态
- 无法判断其他感官是否同时在输出
- 无法决定何时进行认知

因此，Perception 不可能也不应该形成自闭环。

Runtime 作为全局系统组件，具备：

- 对所有感官信号的可见性
- 对系统时间轴的控制权
- 对 Cognition 调用时机的决定权

Perception 的闭环只能在 Runtime 中完成。

---

## 4. Runtime 对 Perception 输出的核心职责

Runtime 对 Perception 输出的处理不涉及语义理解，仅涉及**调度级处理**。

### 4.1 接收（Consume）

- 持续接收来自所有 Perception 的输出信号
- 不要求立即处理
- 不对信号内容做解释

该阶段确保：

> **任何感官输出都会被系统接住，而不是丢失或直达认知层。**

---

### 4.2 缓冲与排队（Buffer / Queue）

- 将并发感官输出转化为可管理的时间序列
- 允许短期堆积
- 允许在策略层面进行合并或丢弃（非语义判断）

该阶段解决的是**并发感知 vs 串行认知**的结构性矛盾。

---

### 4.3 时间窗口化（Windowing）

Runtime 不将单个感官信号直接送入 Cognition。

而是根据：

- 时间关系
- 当前任务状态
- 系统节奏

将多个信号组合成一个**认知时间窗口**。

该窗口代表：

> **“此刻值得被一起理解的一组感官事实”。**

---

### 4.4 触发 Cognition（Triggering）

当 Runtime 判定当前窗口满足认知条件时：

- 触发一次 Cognition 执行
- 将该窗口内的信号作为本轮认知材料

Runtime 决定的是：

- 是否需要认知
- 何时认知
- 使用哪些材料认知

而不是认知内容本身。

---

### 4.5 接收认知反馈（Feedback Loop）

Cognition 的输出会回到 Runtime：

- 是否形成稳定判断
- 是否需要更多输入
- 是否建议执行行动

Runtime 根据这些反馈，调整后续：

- 感官信号的处理节奏
- 认知触发频率
- 行为执行时机

该反馈不会回传给 Perception。

Perception 始终保持“只输出、不感知结果”的特性。

---

## 5. Runtime 形成的系统级闭环

至此，系统在感知—认知路径上形成完整闭环：

Perception → Runtime → Cognition → Runtime →（Action / Memory / 下一轮）

该闭环保证：

- 感官层保持简单与可扩展
- 认知层保持专注与一致
- 系统整体在时间中稳定运行

---

## 6. 设计结论

- Perception 不闭环，是设计正确的结果
- Runtime 是感官信号命运的唯一管理者
- Cognition 只在 Runtime 允许的窗口内发生

通过 Runtime，Agent 获得了类似人类的：

- 感官并发能力
- 注意力与节奏控制
- 渐进式认知与审慎行动

这使得 Agent 不只是“能处理输入”，而是**能在时间中持续理解世界**。
