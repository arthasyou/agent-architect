# Runtime · Internal Structure

> 本文档定义 Runtime 的**内部结构组成**。
> 本文档只描述 Runtime 内部**必须存在的结构单元及其职责边界**，
> 不描述实现细节、调度策略或语义规则。

---

## 1. Runtime 的结构目标

Runtime 是 Agent 的时间与因果裁决系统。

Runtime 的结构目标是：

- 承载并区分不同性质的输入信息
- 在 Epoch 边界上保证因果一致性
- 为裁决提供原子性与可审计性

Runtime **不是**任务调度器、语义系统或策略系统。

---

## 2. Runtime 的整体结构

Runtime 由以下结构单元组成：

```markdown
Runtime
├── Epoch Manager
├── Event Ingress
├── Event Buffering
└── Arbitration Unit
```

上述结构单元 **缺一不可**，
且其职责 **不可合并**。

---

## 3. Epoch Manager

### 3.1 职责

Epoch Manager 负责 Runtime 的时间结构。

其职责包括：

- 维护当前 Epoch 的唯一标识
- 标识当前 Epoch 的 Open / Closed 状态
- 执行 Epoch 的关闭与推进

### 3.2 结构约束

- Epoch Manager **不接收输入事件**
- Epoch Manager **不参与因果裁决**
- Epoch Manager **不接触具体业务数据**

Epoch Manager 仅提供时间结构能力。

---

## 4. Event Ingress

### 4.1 职责

Event Ingress 是 Runtime 的**唯一输入接入点**。

其职责包括：

- 接收外部输入事件（并发安全）
- 为输入事件绑定 Epoch 归属
- 将事件投递至对应的内部缓冲结构

### 4.2 输入类型

Event Ingress 接收以下事件类型：

- Perception Event
- Cognition Result Event
- Action State Event

### 4.3 结构约束

- Event Ingress **不判断事件语义**
- Event Ingress **不触发裁决**
- Event Ingress **不改变事件内容**

Event Ingress 只负责事件接入与归属。

---

## 5. Event Buffering

Event Buffering 负责在 Runtime 内部**承载不同性质的信息**。

Runtime 不存在统一的全局事件队列。

### 5.1 Perception Buffer

#### 职责

- 承载当前 Epoch 内的感知事件
- 支持高并发写入

#### 结构特性

- 不要求事件顺序
- 不保证事件完整保留
- 可在 Epoch 切换时整体废弃

---

### 5.2 Cognition Result Buffer

#### 职责

- 承载已完成、待裁决的认知结果
- 作为裁决候选集合

#### 结构特性

- 异步写入
- 可整体废弃
- 不保证结果被采用

---

### 5.3 Action Registry

#### 职责

- 记录当前存在于时间线中的行为实例
- 支持行为的查询、中断与状态更新

#### 结构特性

- 长生命周期
- 跨 Epoch 存在
- 不以队列形式存在

Action Registry 表示的是**状态集合**，而非事件流。

---

## 6. Arbitration Unit

### 6.1 职责

Arbitration Unit 是 Runtime 中**唯一执行因果裁决的结构单元**。

其职责包括：

- 读取当前 Epoch 的事件快照
- 裁决 Cognition Result 的生效性
- 裁决 Action 的继续、暂停或终止
- 生成跨 Epoch 状态提交指令

### 6.2 结构约束

- Arbitration Unit **只在 Epoch Closed 状态下运行**
- Arbitration Unit **不可并发执行**
- Arbitration Unit **不接收新输入事件**

Arbitration Unit 必须保证裁决的原子性。

---

## 7. 结构单元之间的关系约束

- Event Ingress **依赖** Epoch Manager（获取 Epoch 归属）
- Event Buffering **被** Arbitration Unit 读取
- Arbitration Unit **请求** Epoch Manager 推进 Epoch
- 任意结构单元 **不得反向依赖** Arbitration Unit

Runtime 内部不存在环状依赖。

---

## 8. 非目标（Non-Goals）

Runtime 明确不承担以下职责：

- 输入事件排序或优先级管理
- 语义理解或内容解释
- 行为规划或策略选择
- 资源调度或性能优化

上述职责若出现，应由 Runtime 之外的系统承担。

---

## 9. 结构性保证

通过上述结构划分，Runtime 保证：

- 不同性质的信息被隔离承载
- 因果裁决具有明确边界
- Epoch 推进具备原子性
- Runtime 结构可被直接实现与验证

---

> Runtime 是**结构化的时间系统**，
> 而不是**解释性的控制系统**。
