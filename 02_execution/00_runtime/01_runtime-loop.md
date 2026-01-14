# Runtime Execution Loop（v0.1）

本文档描述 **Runtime 在执行阶段如何形成闭环**：  
它定义“什么时候触发一次 Cognition、如何消费 CognitionResult、如何接入 Action 的反馈、以及何时终止/继续下一轮”。

本文件刻意保持 v0.1 粗粒度：
- 不定义字段细节（字段见 CognitionInput / CognitionResult 文档）
- 不定义具体调度实现（队列/并发/线程模型）
- 不定义工具系统细节（tool schema / tool router）

目标只有一个：**让系统在文档层面成为可运行的闭环**，同时不污染 Cognition 的纯函数边界。

---

## 0. 关键边界（必须成立）

### 0.1 Runtime 不做语义判断

Runtime **不理解**用户语义、不推理、不产出结论。  
Runtime 只做：
- 判断是否需要进入一轮 Cognition（触发条件）
- 把材料交给 Collection/Organize 形成 CognitionInput
- 调用 Cognition 得到 CognitionResult
- 将 CognitionResult 分发到 Action / Memory / Runtime 状态机
- 接收执行反馈（ActionResult / Perception events），决定是否进入下一轮

### 0.2 Cognition 仍然是纯函数

认知层只允许：
`CognitionInput(intent, context) -> CognitionResult(decision, confidence, rationale)`

认知层不负责：
- 调度下一步
- 执行工具
- 写入状态/记忆
- 反向“要更多输入”

相关文档：
- `02_execution/01_cognition_exectuion/03_CongitionInput.md`
- `02_execution/01_cognition_exectuion/02_CognitionResult.md`
- `02_execution/01_cognition_exectuion/01_execution-flow.md`

---

## 1. Runtime 的最小状态机（Minimal FSM）

Runtime 需要一个最小状态机来保证闭环不会“卡住”或“无穷循环”。

建议 v0.1 使用以下状态集合（可裁剪，但不要少于 done + waiting + executing 三类）：

1. `idle`
2. `collecting`
3. `organizing`
4. `thinking`
5. `executing`
6. `waiting`
7. `done`
8. `failed`（可选，但建议保留）

### 1.1 迁移触发（语义层面）

- `idle -> collecting`：出现触发信号（用户输入/系统事件/任务调度）
- `collecting -> organizing`：材料收集完成（得到 Collected Inputs）
- `organizing -> thinking`：组织完成（得到 CognitionInput）
- `thinking -> executing`：CognitionResult 表示需要执行某个意图（见 decision 路由）
- `thinking -> waiting`：CognitionResult 表示需要等待更多信息（例如 request_info）
- `thinking -> done`：CognitionResult 表示本任务已完成（例如 answer / reject 并可终止）
- `executing -> collecting`：ActionResult 产生新材料（执行结果/观测/错误）需要再认知
- `executing -> done`：ActionResult 表示任务完成且无需再认知（策略决定）
- `waiting -> collecting`：等到了外部输入（用户补充/系统事件）或超时触发重试
- `* -> failed`：不可恢复错误（例如 organize 反复失败超过阈值）

注意：Runtime 的“是否需要再认知”决策是基于 **状态与信号**，不是基于语义推理。

---

## 2. 触发一次 Cognition 的条件（Trigger Contract）

Runtime 触发 Cognition 的最小条件是：**当前任务存在一个可判定的问题**。

在 v0.1 里，不要求 Runtime 做复杂判定，推荐使用以下触发原则：

- 有新输入（用户/系统/环境）且当前任务未 `done`
- 当前处于 `executing` 且 ActionResult 返回 `needs_review` / `error` / `partial`
- 当前处于 `waiting` 且满足重试条件（例如超时、或收到了补充信息）
- 启动新任务或切换焦点任务时（task focus change）

触发后进入：
`collecting -> organizing -> thinking`

其中：
- Collection 决定“本轮可见材料边界”（可回放）
- Organize 将材料封装为唯一的 CognitionInput

---

## 3. CognitionResult 的消费方式（Consume Contract）

Runtime 不解释 rationale，只做 **结构化路由**：

### 3.1 decision 路由（建议最小枚举）

为保证闭环可落地，Runtime 至少需要识别这些 decision 语义类型（示例，不强制字段名）：

- `answer`：可直接返回给用户并终止或进入 done
- `plan`：需要执行一组高层意图（交给 Action/Planner，但 Runtime 负责“进入 executing”）
- `route`：需要切换目标子系统/子代理/子任务（Runtime 更新焦点与状态）
- `request_info`：需要用户/外部补充信息（Runtime 进入 waiting，并生成对外请求）
- `reject`：拒绝执行（Runtime 进入 done 或 failed，按策略）
- `noop`：无动作（通常表示输入不足或重复触发，Runtime 可进入 waiting 或 done）

重要约束：
> decision 路由是“流程控制”，不是“语义推理”。

### 3.2 confidence 路由（最小策略）

confidence 的存在意义是让 Runtime 有能力做保守决策。

最小策略建议：

- `high`：按 decision 正常路由
- `medium`：按 decision 路由，但提高审计/复核等级（例如记录更多日志、或要求更严格的 ActionResult）
- `low`：优先进入 `request_info` / fallback / human-in-the-loop（由系统策略选择其一）

v0.1 可以只落一个规则：
> `low` 时不得直接进入不可逆执行（例如外部写操作），必须先等待确认或补充信息。

---

## 4. Action 与反馈回流（Feedback Contract）

闭环成立的关键不是“能执行”，而是“执行后能回流”。

Runtime 需要定义一个最小 ActionResult 语义（字段不必固定，但语义必须有）：

- `status`：`success | partial | error | cancelled`
- `outputs`：本次执行产物（结构化）
- `observations`：环境观测/外部响应（结构化）
- `errors`：错误信息（结构化）

### 4.1 回流规则（最小）

- ActionResult 的 `outputs/observations/errors` 必须进入下一轮 Collection 的材料集合
- ActionResult 的 `status` 决定 Runtime 是否立刻再触发一轮 Cognition：
  - `partial` / `error`：默认触发再认知（collecting）
  - `success`：由策略决定是否再认知（例如需要总结/写记忆时）

---

## 5. Memory 的写入职责（不属于 Cognition）

Cognition 不写 Memory，但系统闭环通常需要“在某些节点写入”。

v0.1 最小约定：
- **写入触发者是 Runtime**（或一个独立的 MemoryManager，由 Runtime 调用）
- **写入依据来自 CognitionResult 与 ActionResult**，而不是来自 Cognition 的隐式副作用

推荐的最小写入时机：
- 任务进入 `done` 时（写入最终结论/摘要）
- ActionResult `success` 且产生可复用产物时（写入事实/配置/路径等）
- 出现 `error` 且定位到稳定原因时（写入故障模式/禁用条件）

---

## 6. 终止条件（Termination Contract）

没有终止条件就不存在闭环，只存在“循环”。

v0.1 的终止条件至少需要覆盖两类：

### 6.1 正常终止

- CognitionResult 表示可以直接结束（如 `answer` / `reject` 且策略允许终止）
- 或 ActionResult 表示完成且无需后续认知（如 `success` 且无待处理事项）

### 6.2 防御性终止

用于避免无穷循环：
- 同一类失败在 `organizing` 或 `thinking` 重复超过阈值
- `waiting` 超时且没有新的外部输入，且重试次数达到上限
- 触发安全策略（例如 low confidence + 不可恢复风险）

Runtime 进入 `failed` 或输出“需要人工介入”的终止状态。

---

## 7. 总流程（闭环视角）

```text
Trigger Signal (User/System/Event)
↓
Runtime: collecting
↓
Collection → Collected Inputs
↓
Runtime: organizing
↓
Organize → CognitionInput
↓
Runtime: thinking
↓
Cognition → CognitionResult
↓
Runtime: route decision + confidence
↓
Action (optional) → ActionResult
↓
Runtime: ingest feedback
↓
(done) OR (back to collecting)
```

---

## 8. v0.1 完成标准（Definition of Done）

当以下问题都有明确答案时，可以认为闭环骨架成立：
- 什么时候触发一次 cognition？
- cognition 的输出如何决定下一步进入 executing / waiting / done？
- executing 的结果如何回到下一轮输入材料？
- low confidence 时系统如何避免不可逆执行？
- 系统如何正常终止？如何防御性终止？

