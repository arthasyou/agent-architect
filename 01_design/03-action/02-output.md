# ActionResult（行动结果）设计文档

## 1. ActionResult 的定位

**ActionResult 是一次 Action 执行后产生的客观事实记录。**

它描述的是：

> **“刚才发生了什么”**
> 而不是
> **“这意味着什么”**

ActionResult 是 Action 对 Runtime 的唯一回应，
也是 Runtime 交还给 Cognition 的**唯一执行反馈来源**。

---

## 2. 角色分工澄清

在 ActionResult 的产生与使用过程中：

- **Action**

  - 执行 ActionInput
  - 收集执行过程中产生的事实
  - 构造并返回 ActionResult

- **Runtime**

  - 接收 ActionResult
  - 记录、调度、转发
  - 不解释结果语义

- **Cognition**

  - 读取 ActionResult
  - 理解其意义
  - 决定是否产生新的行动

因此：

> **ActionResult 是执行事实，不是结论。**

---

## 3. 设计原则

### 3.1 事实优先

ActionResult 只包含**可观察到的事实**：

- 返回值
- 错误信息
- 状态码
- 时间消耗
- 副作用反馈

不做任何主观判断。

---

### 3.2 不做语义归因

ActionResult 不判断：

- 成功是否“符合预期”
- 失败是否“可以接受”
- 是否需要重试
- 是否应该换策略

这些问题全部属于 Cognition。

---

### 3.3 完整而保守

- 宁可多返回原始信息
- 也不提前过滤或压缩“看似无用”的数据

**信息丢失是不可逆的，理解延迟是可接受的。**

---

### 3.4 与 ActionInput 一一对应

一次 ActionInput
→ 产生一次 ActionResult

不合并，不拆分，不跨越。

---

## 4. ActionResult 的本质定义（一句话）

> **ActionResult 是一次副作用执行后留下的世界痕迹。**

---

## 5. ActionResult 最小结构说明（Draft）

### 5.1 核心结构

```
ActionResult {
  action_id
  status
  output
  error
  metadata
}
```

---

### 5.2 字段说明

#### action_id

- 与 ActionInput 中的 action_id 对应
- 由 Runtime 注入并贯穿执行全程

**作用：**
建立输入与输出的严格对应关系
支持调度、追踪与审计

---

#### status

- 描述执行的客观状态
- 不包含价值判断

典型状态包括（示意）：

- success
- failure
- timeout
- cancelled
- unreachable

**作用：**
为 Cognition 提供最粗粒度的执行结果事实

---

#### output

- 执行产生的原始输出
- 结构与具体 Action 实现相关

示例：

- stdout / stderr
- 返回数据
- 设备反馈
- 远程服务响应体

**作用：**
提供最大信息量，供后续理解与推理

---

#### error

- 执行过程中产生的错误信息（若存在）
- 不做归类、不做解释

可能包含：

- 错误码
- 异常描述
- 底层系统信息

**作用：**
作为失败事实的一部分，而非失败原因的解释

---

#### metadata

- 执行过程中的附加事实信息
- 不参与结果语义判断

可能包含（但不限于）：

- 开始 / 结束时间
- 执行耗时
- 使用的执行器信息
- 资源消耗
- 日志索引

**作用：**
支持调度分析、性能评估与审计

---

## 6. 明确不包含的内容

ActionResult **不包含**：

- 行为是否“正确”的判断
- 是否需要重试的建议
- 下一步行动提示
- 世界状态的解释
- 对失败原因的推断

如果这些内容出现，
说明 Action 或 Runtime 越权了。

---

## 7. ActionResult 在闭环中的位置

ActionResult 本身**不形成闭环**。

完整执行闭环如下：

```
Perception → Cognition → Runtime → Action
                               ↓
                         ActionResult
                               ↓
                         Runtime → Cognition
```

ActionResult 只是一次“回声”，
闭环的理解发生在 Cognition 中。

---

## 8. 核心共识（一句话）

> **ActionResult 是事实，不是答案。**

它不告诉系统“该怎么办”，
只告诉系统“刚才发生了什么”。
