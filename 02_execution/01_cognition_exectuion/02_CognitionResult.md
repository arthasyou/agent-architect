#

## CognitionResult 的最小结构（Minimal）

**CognitionResult = 一次认知决策的不可再分结果**

它至少只需要 **三块**：

```rust
CognitionResult {
  decision
  confidence
  rationale
}
```

### 1. decision（必须）

**这是唯一不可删除的字段。**

它代表：

> 认知层最终“认为该怎么做 / 该认为什么”

形式不重要，但必须是**可被下游系统消费的结构化结果**，而不是自然语言。

例子（仅示意）：

```ts
decision: {
  type: "plan" | "answer" | "route" | "reject"
  payload: {...}
}
```

如果没有 decision，认知层就没发生“决策”，那它只是个 tokenizer。

---

### 2. confidence（强烈建议，但仍属最小）

这是认知层对**自身决策稳定性的声明**。

不是概率论意义的置信区间，而是工程意义上的：

- 我确信
- 我勉强判断
- 我其实不确定

最小可压缩为：

```ts
confidence: low | medium | high;
```

为什么它属于“最小”而不是“增强”？

因为 **没有 confidence，下游无法判断是否需要：**

- 再问一次
- 触发 fallback
- 升级到人类
- 进入 review 流程

没有它，系统只能假装 LLM 永远正确，这是工程灾难。

---

### 3. rationale（最小可审计解释）

不是“思维链”，而是**结果级解释**。

一句话定义：

> rationale = 我为什么给出这个 decision（供系统理解，不供模型推理）

最小形态可以是：

```ts
rationale: string;
```

或结构化版本：

```ts
rationale: {
  basis: [key_facts];
  constraints: [applied_rules];
}
```

可以把它想成：  
**“如果这个结果出问题，我该从哪儿查起”**

如果完全不关心可审计性，这个字段可以删。  
但一旦想 debug、对 CTO 解释、或做风控，它立刻变成刚需。

---

## 为什么不需要更多？

下面这些**现在都不属于 CognitionResult 的最小结构**：

- ❌ action（那是 execution 层的事）
- ❌ memory diff（那是 memory 层的事）
- ❌ world model（那是认知输入，不是输出）
- ❌ next step（那是 planner 的事）
- ❌ tool call（那是 runtime 的事）

**CognitionResult 只回答一件事：**

> “在当前上下文下，我的判断结果是什么？”

---

## 一句话版本

> CognitionResult 是认知层的最小、原子化输出，  
> 它描述了一次可消费、可评估、可追溯的决策结果，  
> 而不包含任何执行、状态变更或系统副作用。
