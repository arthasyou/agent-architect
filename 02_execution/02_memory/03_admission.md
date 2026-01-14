## Memory Admission Specification

### 1. 目的

本文档定义 **Memory Admission（写入准入层）** 的职责、接口与约束。

Memory Admission 位于 Cognition 与 Memory Storage 之间，
用于确保所有进入 Memory 的记录：

- 结构合法
- 权限允许
- 资源可承载

Memory Admission **不参与任何语义判断**，
不决定记录是否“有价值”“重要”或“重复”。

---

### 2. 角色定位

Memory Admission 是一个**规则执行层**，而非决策层。

其职责是：

> **在不理解内容的前提下，
> 决定一次写入请求是否可以被接受。**

Admission 不产生新的信息，
也不修改写入内容的语义。

---

### 3. 输入与输出

#### 3.1 输入：Memory Write Candidate

Admission 接收的输入是一个 **写入候选（Write Candidate）**，由 Cognition 构造。

写入候选必须包含：

- `record`
  一个完整的 Memory Record（符合 Record Model）

- `source`
  写入来源标识（如 Cognition 实例、阶段标识）

- `intent`（可选）
  写入发生的认知意图标识，仅用于审计，不参与判断

Admission 不关心 Record 的内容含义。

---

#### 3.2 输出：Admission Result

Admission 的输出必须是以下之一：

- **ACCEPT**
- **REJECT**

当返回 REJECT 时，必须附带明确的拒绝原因码。

Admission 不返回“部分成功”“自动修正”等中间状态。

---

### 4. Admission 的判断范围

Admission **只允许**基于以下三类约束做出判断。

---

#### 4.1 结构合法性校验（Structural Validation）

Admission 必须校验：

- Record 类型是否被允许
- Record 是否符合对应类型的结构约束
- 必填字段是否存在
- 不可变字段是否被篡改
- 是否包含被禁止的数据形态

不通过结构校验的写入 **必须被拒绝**。

---

#### 4.2 资源约束校验（Resource Constraints）

Admission 必须校验：

- Memory 是否仍有可用容量
- 是否超过单次或单位时间内的写入配额
- 是否违反存储上限或安全限制

当资源不足时，Admission **必须拒绝写入**，
不得触发自动清理或重组。

---

#### 4.3 权限与生命周期校验（Authorization & Lifecycle）

Admission 必须校验：

- 写入来源是否具备写入权限
- 当前 Agent 是否处于允许写入 Memory 的生命周期阶段
- 写入是否违反系统级安全或隔离规则

权限不满足时，必须拒绝。

---

### 5. Admission 明确不做的事情

Memory Admission **不得**：

- 判断记录是否重复
- 判断记录是否重要或有价值
- 判断记录是否“已经存在”
- 合并、压缩或修改记录内容
- 生成摘要或派生信息
- 基于语义做任何推断

Admission 是**无语义层**。

---

### 6. 拒绝语义（Failure Semantics）

写入被拒绝是 **正常且可预期的系统行为**。

Admission 的拒绝应满足：

- 明确：有确定的拒绝原因码
- 稳定：相同条件下结果一致
- 可审计：拒绝事件可被记录为 Event（可选）

拒绝不应被视为异常或错误状态。

---

### 7. Admission 与 Memory Storage 的关系

Admission 不直接操作存储介质。

当且仅当 Admission 返回 ACCEPT 时：

- Record 才会被提交给 Memory Storage
- Storage 不再进行任何额外判断

Memory Storage 假定 Admission 的判断结果是最终且可信的。

---

### 8. 设计自检规则

在为 Admission 增加任何新规则前，必须回答：

- 该规则是否依赖对内容“意义”的理解？
- 该规则是否在替 Cognition 做判断？
- 该规则是否在隐式决定“什么值得被记住”？

若答案为是，则该规则 **不应存在于 Admission 中**。

---

### 9. 总结性定义

Memory Admission 是：

- 冷静的
- 被动的
- 无语义的
- 可拒绝的

**写入守门层**。

它的存在不是为了让 Memory 更聪明，
而是为了确保 **Memory 永远不会变聪明**。

---

### 一句话总结整条链路（非常重要）

> Cognition 决定 _要不要记_，
> Admission 决定 _能不能记_，
> Memory 负责 _记或拒绝_。
