# Analyse 目标

维护一份简洁、可更新的结构分析摘要，让后续审阅直接从共同基线出发，避免重复推导核心边界。

## 当前分析快照

**核心结构（6 个功能）**
- 感官（Perception）：只接收外部输入，输出统一、低语义信号。
- 认知（Cognition）：组织输入为可判定问题并输出结构化决策。
- 行动（Action）：执行副作用，返回事实结果，不做判断。
- 时间/因果（Runtime）：维持单一时间线并裁决何时进入现实。
- 记忆（Memory）：只保存长期事实/历史，不做语义判断或自动整理。
- 整体状态（Agent State）：派生的全局运行背景，只读不控。

**关键边界**
- 感官不解释语义；来源标记只供 Runtime/Collection 使用。
- Collection/Organize 是唯一生成 CognitionInput 且可读取 Memory 的路径。
- Cognition 是对 CognitionInput 的纯函数，不直接访问 Memory。
- Runtime 裁决时间线有效性并生成 ActionInput；Action 不做决策。
- Memory 写入仅源自 Cognition；Admission 只做结构校验。
- Agent State 为派生状态，不由任何模块显式设置。

**Runtime 模型**
- 基于 Epoch 的因果时间；Open/Closed 两态；裁决仅在 Close 执行。
- 输入为事件（Perception/CognitionResult/ActionState），而非查询。
- 输出为裁决生效（Action 生命周期、Epoch 推进、状态提交）。

**Memory 模型**
- 存储为可读记录（FACT/EVENT/STATE）。
- 不以向量、token、摘要或派生指标为权威。
- IO 读取有界；读取只由 Collection 发起；写入由 Cognition 发起。

**执行闭环（高层）**
感官 → Runtime → Collection/Organize → 认知 → Runtime → 行动 → Runtime →（循环）
Memory 通过 Collection 读、Cognition 写参与闭环；Agent State 为 Cognition/Runtime/Action 的全局背景。

## 开放问题（如有）
- 当前无记录。

## 最后更新
- 2025-01-17
