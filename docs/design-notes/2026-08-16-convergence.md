# 2026-08-16 Design Convergence

> **Type:** Design note / historical record  
> **Purpose:** 记录今天多轮讨论为什么看起来反复“绕回来”，以及最终如何把 Personal Context、DSH、GA、MemOS、数据库和 Personal DSH Distribution 放到同一套层次里。  
>
> 这份文档保存演化过程，不作为 architecture spec。当前状态以 `design-state.md` 为准。

---

## 1. 背景

今天的讨论实际上叠加了几条不同来源的思路：

1. 2026-04-07 的原始 Personal AI OS / 长期个人数据构想；
2. 对 DSH、GenericAgent、MemOS、OpenViking、Graphiti、TencentDB Agent Memory 等现有系统的生态审计；
3. State / Claim / Skill、Evidence、Proposal、Governance、Environment Compiler 的架构讨论；
4. 实际使用 GenericAgent 后对 Markdown / 模型自主读取可靠性的反思；
5. DSH “everything is a plugin” 与 Profile / Bundle 让 Personal AI Distribution 突然变得现实；
6. 对 README、创新点和宣传点的多轮重新表述。

表面上看，项目在：

```text
Personal Context Layer
↔ Personal DSH Distribution
↔ Memory / Learning system
```

之间反复摆动。

最终判断是：

> 这些并不是互斥的项目定义，而是不同层。

---

## 2. 今天最重要的纠偏：不要让最新材料重写全部设计

此前讨论中出现了一个明显问题：

- 一拿到 DSH + GA 材料，就把项目重定义成 Personal DSH；
- 一拿到 MemOS / Context 材料，又把项目重定义成 user-owned Context Layer；
- 新的竞品功能又会让宣传点整体重排。

这种方法不稳定。

正确方式应该是先冻结：

```text
stable principles
current baseline
hypotheses
open questions
```

之后新材料只修改对应局部。

这也是 `design-state.md` 出现的原因。

---

## 3. Personal Context 与 Personal DSH 的最终层次

### Core

```text
Personal Context
├── structured data model
├── provenance / authority
├── lifecycle
├── update proposals
├── governance
├── context assembly
├── disclosure
└── UI
```

这是当前核心自研问题。

### First runtime

```text
DSH
```

DSH 提供：

- runtime；
- session；
- tools；
- plugins；
- storage primitives；
- approval；
- sandbox；
- UI extension；
- Profile / Bundle / Preset。

### Distribution / delivery

未来可以形成：

```text
Personal DSH Distribution
├── Personal Context
├── Memory Provider
├── GA-inspired evolver
├── Skills
├── domain packs
├── adapters
└── presets
```

所以：

> **Personal DSH 是很自然的产品化与 dogfood 形态，但不替代 Personal Context Core 的问题定义。**

---

## 4. 数据库从“实现细节”上升为稳定原则

今天重新确认了一个原始想法中一直存在、但后来被架构术语稀释的要求：

> **肯定不能只靠 Markdown 文件和模型自己维护长期认知。**

GenericAgent 的实际体验强化了这一点。

GA 的设计非常优雅：

```text
tiny textual index
→ LLM realizes something exists
→ LLM reads deeper Markdown
→ LLM interprets
→ LLM uses it
```

问题不在 Markdown 文件格式本身，而在：

> retrieval policy、schema semantics、authority、ACL、state lifecycle 等关键控制职责如果都隐含在文本里，就过度依赖模型能力。

因此冻结：

> **控制面结构化，内容面自由。**

数据库技术仍未决定。

---

## 5. DSH Session 的正确位置

DSH Session Log 不需要被复制成第二套 conversation DB。

它最有价值的位置是：

# Evidence Ledger

例如：

```text
State:
repository.remote = none

Evidence:
DSH session X / seq Y
tool: git remote -v
result: empty
```

必须坚持：

> **Evidence ≠ Truth**

Session Log 提供证据，不自动决定权威状态。

---

## 6. GA 的正确位置

今天一度把项目向“GA-inspired Personal Evolution Bundle”拉得过远。

最后收敛为：

### 保留

- Contextual Information Density；
- Experience → Pattern；
- Pattern → SOP / Skill；
- 长期 Learning Loop。

### 不直接继承

- Markdown 事实库作为 Canonical State；
- LLM 自己决定什么时候读取关键个人事实；
- 模型自己维护长期权威状态；
- Experience 自动变成 Truth。

因此可以设计：

```text
experience-evolver
skill-evolver
```

但它们属于 Personal Context / Personal DSH 的学习子系统，而不是全部项目。

---

## 7. MemOS 的正确位置

一度把以下能力当成 Personal Context 的宣传差异：

- context belongs to user；
- cross-agent memory；
- Memory Viewer。

重新对照 MemOS 后确认：

> 这些能力本身已经不能作为主要差异化。

MemOS 当前定位：

```text
important reference
+ reusable Memory / Learning Provider
+ baseline
```

Personal Context 不需要重写其擅长的长期 Memory / Skill 学习。

真正的上层边界是：

```text
MemOS-derived understanding
→ Evidence / candidate / proposal
→ Personal Context governance
→ authoritative long-term assets
```

---

## 8. 重新确认的核心架构

### Long-term assets

```text
State
Claim
Skill
```

### Shared evidence layer

```text
Evidence
```

### Write

```text
Evidence
→ Proposal
→ Governance
→ Commit
```

### Read

```text
Activity
→ candidate retrieval
→ type-aware assembly
→ budget
→ deterministic disclosure
→ Agent
```

### Human

```text
browse
edit
review
trace
audit
export
delete
```

---

## 9. Runtime Compiler 是新增 Hypothesis，不是新核心定义

GA 的真实问题提示：

> 模型即使“看见”正确信息，也可能不会正确使用。

于是出现：

```text
Context Compiler
→ what the model sees

Runtime Compiler
→ what the agent can use
```

例如根据环境自动：

- 暴露对应 Skill；
- 隐藏不兼容 Skill；
- 配置 Tool defaults；
- 修改 Preset。

这个方向很有潜力，但当前不能因为它很漂亮就重写整个项目。

它必须通过：

- weaker model；
- fewer token；
- fewer tool calls；
- stable success；

来证明。

---

## 10. 外部 App 联邦的叙事被收缩

此前曾经把：

> “原始数据继续留在 Source of Truth，Personal Context 只做引用和连接”

讲得过于理想化。

现实是大量闭源 App：

- 无 API；
- 跨设备；
- 数据结构不透明；
- 自动化困难；
- 无稳定 export。

因此新的边界：

> **External adapters 是增量能力，不是 Personal Context 成立的前提。**

第一阶段先理解 Agent 自己可靠可观察的世界：

- DSH Session；
- Tool results；
- Git；
- Files；
- workspace；
- system probes；
- explicit user input。

---

## 11. 明确回退 / 不再采用的宣传与定位

### 不再把这些当主要创新

- user-owned context；
- cross-AI sharing；
- Memory Viewer；
- hierarchical context；
- hybrid retrieval；
- temporal facts；
- ACL；
- Skill evolution。

它们仍然可以是产品能力，但不是独有创新。

### 不再把 Personal DSH Distribution 当成 Core 的替代定义

Distribution 是组合和交付层。

### 不再假设“所有现有 App 都容易连接”

Adapter 必须面对现实可用性。

### 不再接受纯 Markdown 作为 authoritative control plane

Markdown 继续是优秀内容格式，不是数据库 / ACL / state machine。

### 不再允许主 LLM 直接写长期权威状态

默认必须走 Proposal / Governance。

### 不再把确定性理解成 if/else relevance routing

确定的是：

- policy；
- scope；
- disclosure；
- lifecycle constraints。

相关性可以模型化。

---

## 12. 当前最值得验证的创新假设

不是单个算法，而是组合：

```text
structured DB control plane
+
State / Claim / Skill decision semantics
+
Evidence-backed write governance
+
lifecycle / stale / revalidation
+
type-aware Context Assembly
+
deterministic disclosure
+
single final assembly owner
+
provider federation
+
human-visible governance
```

是否真的值得存在，要用 baseline 证明。

---

## 13. 第一阶段应该做什么

先做一个窄 vertical slice：

# Device & Project Environment

验证：

```text
State
→ Context
→ Agent action
→ Tool Evidence
→ Proposal
→ Governance
→ updated State
```

并做 UI：

- State；
- Evidence；
- stale；
- Proposal；
- Context Run。

Memory / Skill Evolution 可以作为第二阶段接入。

---

## 14. 下一步文档策略

从今天起：

- `origin.md`：只记录为什么开始；
- `vision.md`：只记录最终想达到什么；
- `principles.md`：只记录稳定原则；
- `design-state.md`：成为当前 canonical state；
- `architecture.md`：当前 baseline；
- `open-questions.md`：尚未决定；
- `related-work.md`：已有系统解决了什么；
- `design-notes/`：保留历史演化。

后续新讨论不应该再次把聊天记录当作唯一“数据库”。

如果某个新想法与当前设计冲突，先问：

1. 它推翻了 Stable Principle 吗？
2. 它只是替换 Baseline 吗？
3. 它只是一个新 Hypothesis 吗？
4. 还是应当进入 Open Questions？

只有第 1 种情况才需要重新审视整个项目。
