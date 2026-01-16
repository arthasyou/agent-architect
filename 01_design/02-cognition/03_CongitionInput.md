**CognitionInput 的目标不是“信息齐全”，而是“可判定”。**

换句话说：

> 只要能让认知层**做出一次确定的 decision**，它就是“足够的输入”。

---

## CognitionInput 的最小结构（Minimal）

直接给下限结构：

```rust
CognitionInput {
  intent
  context
}
```

没有第三项。现在先别加。

---

### 1. intent（不可删除）

**intent = 这一次认知“要解决什么问题”**

它不是用户原话，也不是 prompt，而是**已经被上游系统收敛过的语义目标**。

最小形态：

```
intent: {
  type
  payload
}
```

例子（示意）：

- type: "answer_question"
- type: "make_decision"
- type: "plan_steps"
- type: "judge_feasibility"

**关键点**：
intent 必须是**单一的**。
如果一次输入里有多个 intent，那是 orchestration 的失败，不是 cognition 的问题。

---

### 2. context（不可删除）

**context = 做出判断所需的已知条件**

它是“事实集合”，不是“世界模型”。

最小可以极端到这样：

```
context: {
  facts: [...]
}
```

facts 可以是任何结构化片段：

- 用户输入
- 系统状态
- 历史摘要
- 环境信号

**重要边界**：

- context 是**只读的**
- context 不承诺完整
- context 只对本次 decision 有效

认知层**不能假设** context 覆盖了世界，只能假设“这是我被允许知道的全部”。

---

## 为什么不需要更多？

现在**刻意不加**这些东西：

- ❌ memory（那是 context 的来源，不是 input 字段）
- ❌ constraints（规则也是事实的一种）
- ❌ tools（认知层不关心能不能做，只关心“该不该”）
- ❌ time（时间是 context 的一个 fact）
- ❌ priority（优先级是 intent 的属性）

你现在要的是**认知函数**，不是智能体人格。

---

## 输入 → 输出的最小闭环

```
CognitionInput(intent, context)
        ↓
   Cognition Layer
   (single LLM inference)
        ↓
CognitionResult(decision, confidence, rationale)
```

这是一个**纯函数**视角：

- 无副作用
- 无状态写入
- 无隐式依赖

认知层在最小实现下**可以直接由一次 LLM 推理完成**，
中间不要求拆分为多个子阶段或组件。

一旦你发现某个字段“必须写回系统”，
那它就**不应出现在 CognitionInput / CognitionResult 中**，
而应由执行层或状态管理层处理。

---

## 一个判断标准（非常重要）

以后每当你想给 CognitionInput 加字段，问一句：

> **如果我删掉它，认知层还能不能“被迫给出一个判断”？**

- 能 → 不该加
- 不能 → 才有资格进入最小结构
