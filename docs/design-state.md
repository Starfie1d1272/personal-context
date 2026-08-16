# Current Design State

> **Status:** Canonical working snapshot  
> **As of:** 2026-08-16  
> **Purpose:** 将截至目前多轮讨论中已经收敛的结论、当前默认架构、研究假设和开放问题分开记录。  
> **Rule:** 后续新讨论默认只修改本文件中的局部状态，不重新从零定义整个项目。  
>
> `origin.md` 继续负责记录项目为什么出现；本文不覆盖或改写项目起源。

---

## 1. 当前一句话定义

**Personal Context 是一个面向长期 AI 协作的、数据库支撑的用户自主认知与上下文基础设施实验。**

它不试图再造一个 Agent Runtime、Memory OS 或 RAG 系统，而是研究如何把关于用户、项目和现实环境的长期信息以明确语义维护为可查询、可追溯、可治理、可更新的长期资产，并在每次活动中为 Agent 生成真正需要且允许披露的运行环境。

当前最重要的区别不是“记住更多”，而是：

- 当前状态是否仍然正确；
- 一个判断凭什么成立；
- 一条经验在什么条件下应当转化成可复用能力；
- 新信息是否有资格写入长期认知；
- 这一次 Agent 到底应该看到什么；
- 用户能否直接看到、修改和审计这些长期认识。

---

## 2. 状态标签

本文使用以下标签：

- **Stable**：当前认为已经足够稳定，除非出现新证据或明确设计决议，不应被普通实现讨论反复推翻。
- **Baseline**：当前默认实现方向，可以调整，但新方案需要说明为什么更好。
- **Hypothesis**：有价值但尚未被实验或长期 dogfood 证明。
- **Open**：尚未决定。
- **Rejected / Deprioritized**：已经明确不应作为当前核心方向，或至少第一阶段不做。

---

## 3. Stable：已经锁定的核心结论

### 3.1 长期认知的控制面必须结构化

Personal Context 的核心长期资产不能主要依赖 Markdown 文件和 LLM 自主管理。

需要一个 **schema-backed、tool-mediated、可查询、可约束、可版本化、可追溯的数据库/control plane**。

这并不意味着所有内容都必须进入数据库：

- State、Claim 元数据、EvidenceRef、Proposal、Policy、Scope、关系和生命周期信息适合结构化保存；
- SOP、Skill 说明、Episode、长文本、原始文档和附件仍然可以保留为 Markdown、文件、Blob 或外部 Provider。

当前稳定原则是：

> **控制面结构化，内容面自由。**

具体数据库技术仍未锁定。

### 3.2 Memory 是基础能力，不是最高层语义

Memory 可以负责保存、压缩、关联、召回长期信息，也可以成为重要 Provider。

但不能把当前状态、历史经历、科学主张、用户偏好和 Skills 全部重新压成一个统一的：

```text
memory item = text + timestamp + vector + importance
```

Personal Context 需要在 Memory 之上保留明确的长期决策语义。

### 3.3 State / Claim / Skill 按决策语义区分

当前核心模型至少区分三类长期资产：

- **State**：Agent 据此判断“现在是什么”；
- **Claim**：Agent 据此判断“我们有什么理由相信什么”；
- **Skill**：Agent 据此判断“在什么条件下应该怎么做”。

区别不在三个名字本身，而在错误和冲突发生后治理方式不同：

```text
State 冲突
→ 解析当前有效、权威的值

Claim 冲突
→ 允许并存，保留 support / contradict / qualify / scope

Skill 失败
→ 修改适用条件、步骤、验证方式或可靠性
```

这套分类仍然是 **design thesis**，最终是否值得保留必须通过实验验证；但在当前设计中它是默认语义基线。

### 3.4 Evidence 不等于事实

所有长期认识都应能够追溯到 Evidence。

Evidence 可以来自：

- 用户消息；
- Agent 会话；
- Tool call / Tool result；
- DSH Session Log；
- Git / 文件 /系统探针；
- 网页、论文、数据集；
- 外部 API；
- 一次完整任务轨迹。

Evidence 只回答：

> **“凭什么产生这个判断？”**

它本身不自动成为 State、Claim 或 Skill。

DSH Session Log 因此非常适合作为重要的 **Evidence Ledger**，但不能直接充当“真相数据库”。

### 3.5 长期写入必须经过 Evidence → Proposal → Governance → Commit

当前稳定写入模型：

```text
Observation / Conversation / Tool result
                ↓
             Evidence
                ↓
      Candidate interpretation
                ↓
             Proposal
                ↓
  authority / conflict / risk / policy
                ↓
            Governance
                ↓
              Commit
```

LLM 可以提出“世界可能变了”，但不天然拥有修改长期权威状态的权力。

不同 Evidence Source 可以拥有不同的 authority 和自动写入能力；具体等级仍在探索。

### 3.6 长期信息必须有生命周期，而不是只增长

State 和其它长期资产需要显式考虑：

- created / observed / verified time；
- valid_from / valid_until；
- freshness；
- superseded / retired；
- stale / needs_revalidation；
- 来源权威性；
- 历史版本。

“记住了”不等于“永远继续影响未来 Agent”。

### 3.7 读取不是 Search → Prompt，而是 Context Assembly / Compilation

检索只是候选生成的一部分。

当前稳定的读取思想是：

```text
Activity
  ↓
Scope / hard constraints
  ↓
Candidate retrieval
  ↓
Fusion / rerank
  ↓
Type-aware handling
  ↓
Budget
  ↓
Disclosure policy
  ↓
Serialization
  ↓
Agent environment
```

其中：

> **相关性可以是概率的，权力边界必须是确定的。**

Embedding、BM25、reranker、小模型甚至 LLM 可以判断“是否相关”；但权限、隐私、撤销、过期、允许给哪个模型看等边界不能依赖主模型临场判断。

### 3.8 Provider 可以很多，但最终上下文组装责任必须唯一

未来可以同时存在：

- 结构化 Personal Context DB；
- MemOS；
- OpenViking；
- Graphiti；
- DSH Session；
- Files / Git / Zotero；
- Skills；
- 其它 Provider。

Provider 可以提供候选，但不应每个插件都独立把内容偷偷 inject 到 Prompt。

一次 Agent invocation 最终必须有一个明确的 **Context Assembly & Disclosure Owner**，能够解释：

- 为什么选中；
- 为什么没选中；
- 来自哪里；
- 是否当前有效；
- 为什么允许披露；
- 最终给了 Agent 什么。

### 3.9 用户界面是系统要求，不是附加 Viewer

任何进入 Personal Context 长期认知范围的内容都应该对用户可见、可查询和可追溯。

用户至少需要能够：

- 浏览 State；
- 查看 Claims；
- 管理 Skills；
- 修改信息；
- 查看历史和冲突；
- 审核 Proposals；
- 查看 Evidence / provenance；
- 查看哪些 Agent / Model 可以访问；
- 导出和删除；
- 查看一次 Agent Context Run 实际包含和排除了什么。

两个核心问题必须能够直接回答：

> **“你为什么知道这件事？”**

> **“为什么这一次 Agent 能 / 不能看到它？”**

### 3.10 DSH 是第一 Runtime，不是数据所有者

第一阶段使用 DSH 作为主要 Runtime 和 reference implementation，复用其：

- Session；
- Agent lifecycle；
- Tools；
- Skills；
- Storage；
- Permissions / Approval；
- Sandbox；
- Web UI / client extension；
- Profile / Bundle / Preset；
- MCP / external integration；
- subagent / workflow 等。

但 Personal Context 的核心语义和数据模型不能依赖 DSH 的私有实现。

长期应允许：

```text
Personal Context Core
├── DSH
├── MCP
├── other agent runtimes
└── future adapters
```

### 3.11 Event-driven 优先于 autonomous heartbeat

系统不应为了“保持智能”持续让 LLM 醒来思考。

优先由以下触发：

- 用户输入；
- Tool / system event；
- 外部事件；
- 明确 schedule；
- 数据变化；
- 必要的 revalidation。

默认是：

```text
event → 必要处理 → 结束
```

而不是持续 heartbeat → LLM。

### 3.12 稳定核心语义，开放领域扩展

核心不尝试设计覆盖整个现实世界的万能 ontology。

稳定的是少量基础概念，例如：

- Entity；
- State；
- Claim；
- Skill；
- EvidenceRef；
- Proposal；
- Relation；
- Policy；
- Subject；
- Scope。

科研、软件工程、设备、健康、资产等领域通过扩展 schema / domain pack 表达。

---

## 4. Baseline：当前默认架构

### 4.1 控制面与内容面

```text
Structured control plane
├── Entities
├── States
├── Claims
├── Skill metadata
├── Evidence references
├── Proposals
├── Relations
├── Policies
├── Scopes
├── lifecycle / revision metadata
└── Agent Context Runs

Content plane
├── Markdown SOP
├── Skill instructions
├── executable code
├── Episode / summaries
├── raw files
├── documents
├── DSH Session Log
└── external providers
```

### 4.2 两条主要长期闭环

#### Reality / Truth Loop

```text
Observation
   ↓
Evidence
   ↓
State Proposal
   ↓
Verification / Governance
   ↓
Canonical State
   ↓
revalidation / supersession / retirement
```

#### Learning Loop

```text
Session / Experience
   ↓
Pattern
   ↓
Skill Candidate
   ↓
Verification / Evaluation
   ↓
Skill
   ↓
Reuse
   ↓
new Experience
```

两条 Loop 可以共享 Evidence 和工具，但不能把 Experience 自动“晋升”为 Truth。

### 4.3 Context Compiler

Context Compiler 负责：

> **模型这一次需要看到什么？**

默认输入包括：

- 当前活动；
- 用户 / workspace / project；
- 当前 Agent / preset；
- 当前模型；
- 当前工具；
- token / latency budget；
- Personal Context；
- Memory / Knowledge / Skill providers。

### 4.4 第一实现优先处理 Current State

第一阶段优先验证最清晰的闭环：

```text
State
→ 按需 Context
→ Agent action
→ new Evidence
→ Proposal
→ Governance
→ State update
```

长期 Memory / Skill Evolution 可以随后接入，避免第一个实验退化成“比较哪个 Memory Engine recall 更好”。

---

## 5. Hypothesis：当前值得探索但尚未锁定

### 5.1 Runtime Compiler

除了 Context Compiler，还可能需要：

```text
Context Compiler
→ 模型应该看到什么

Runtime Compiler
→ Agent 应该拥有什么
   tools / skills / preset / runtime defaults / policy
```

目标是把一部分原本依赖模型“读懂并记得使用”的智能下沉到 Harness。

例：

```text
os = macOS
package_manager = pnpm

→ 加载 macOS / pnpm Skill
→ 不加载 Windows-only Skill
→ package-management tool 使用对应 defaults
```

这是当前很有潜力的方向，但尚未证明必须独立成为 Compiler。

### 5.2 分级治理

如果所有 Proposal 都要求人工确认，系统不可长期使用。

候选模型：

```text
auto-commit
silent proposal
batch review
immediate approval
never infer
```

如何按 authority、risk、sensitivity、reversibility、confidence 分级，仍需 dogfood。

### 5.3 Provider abstraction

Memory、State、Graph、Documents 等是否需要统一 provider contract，以及 contract 应多薄，尚未决定。

### 5.4 Personal DSH Distribution

基于 DSH 的 Profile / Bundle 做一套 Personal AI “整合包”是很合理的交付和 dogfood 方式。

但当前层次判断是：

- **Personal Context**：核心自研语义 / control plane；
- **Personal DSH Distribution**：可能的第一产品化 / 组合发行方式。

Distribution 不替代 Core 的项目身份。

---

## 6. 现有系统的位置

### DSH

**角色：** 第一 Runtime / composition substrate。  
**不重做：** Agent loop、session、tools、storage framework、skills runtime、permissions、sandbox 等。  
**核心用法：** Session Log 作为 Evidence 来源，pre-step / service / storage / UI extension 作为实现落点。

### GenericAgent

**角色：** 重要设计参考，尤其是：

- Contextual Information Density；
- Experience → SOP / code / Skill；
- 长期 Learning Loop。

不直接继承：

- 让 Markdown 同时承担状态数据库职责；
- 依赖 LLM 自己意识到何时读取所有关键长期事实；
- 让 LLM 自主管理权威状态。

### MemOS

**角色：** 重要 Memory / Learning Provider 候选 + baseline + reference implementation。

可复用其长期记忆、经验抽取、策略 / Skill 演化等能力。

MemOS 产生的“世界认识”进入 Personal Context 时，默认应视为 Evidence / Candidate / Proposal，而不是无条件成为 authoritative State。

### OpenViking

**角色：** Context Database / hierarchical retrieval 的重要 baseline 或 Provider。

不重做其已经成熟的分层组织、retrieval / rerank 思路；重点比较其输出进入长期治理层后仍存在什么问题。

### Graphiti

**角色：** temporal facts / provenance / graph provider 或 baseline。

不把“有时间和 provenance 的 State”本身当作创新；重点研究 authoritative current state、Claim semantics 与最终 Context 正确性。

### TencentDB Agent Memory

**角色：** ACL / asset management / Agent loadout / hybrid retrieval 的重要对照。

不把“有权限、给不同 Agent 配不同知识”本身当作创新；继续研究更细粒度 disclosure 和最终 Context responsibility。

---

## 7. 当前明确不是创新的单点

当前不应单独宣传为项目创新：

- 长期 Memory；
- 跨 Session recall；
- 分层 Context；
- hybrid retrieval；
- embedding / reranker；
- temporal facts；
- provenance；
- ACL；
- Agent loadout；
- Experience → Skill；
- Context injection；
- Memory Viewer；
- “不同 Agent 可以共享 Memory”。

当前真正值得验证的是这些能力之上的组合架构：

```text
structured long-term control plane
+ decision-semantic State / Claim / Skill
+ evidence-backed write governance
+ lifecycle / freshness
+ type-aware context assembly
+ deterministic disclosure
+ provider federation
+ user-visible auditability
```

这应称为 **architectural hypothesis / design thesis**，不是“世界首创”。

---

## 8. 外部应用与 Source of Truth 的现实边界

长期愿景允许 Git、Zotero、Finance、Home Inventory、Calendar 等成为外部 Source / Provider。

但当前不假设：

> “所有闭源 App 都能轻易接入 Personal Context。”

现实中很多应用：

- 没有 API；
- 数据在不同设备；
- schema 不透明；
- 自动化受限；
- 导出不稳定；
- 只有 UI。

因此外部 Adapter 是增量能力，不是系统成立的前提。

第一阶段首先依赖 Agent 自己能够可靠观察的世界：

- DSH Session；
- Tool results；
- workspace / files；
- Git；
- shell / system probes；
- user explicit input；
- 少量高质量开放 integration。

---

## 9. 第一原型

优先场景：

# 设备与项目运行环境

最小闭环：

```text
DSH Session / system probe
        ↓
      Evidence
        ↓
  Personal Context DB
        ↓
   Context Compiler
        ↓
       Agent
        ↓
   new Tool Evidence
        ↓
      Proposal
        ↓
  user / policy review
        ↓
       Commit
```

需要展示：

- 当前 device / OS / shell / project environment；
- provenance；
- freshness；
- Context Run included / excluded；
- 状态变更 Proposal；
- Accept / edit / reject；
- history。

---

## 10. 评估与可证伪性

第一阶段至少比较：

```text
A. DSH only

B. DSH + mature Memory / retrieval baseline

C. DSH + baseline
   + structured current state
   + evidence-backed write governance
   + type-aware context assembly
   + disclosure policy
```

关注指标：

- 用户重复解释背景的次数；
- Context token；
- stale / wrong State 注入次数；
- 错误长期写入次数；
- provenance coverage；
- 不必要敏感信息披露；
- 第二次完成类似任务的 token / tool calls / success rate；
- 用户维护成本；
- 更弱 / 更便宜模型能否复用已沉淀能力。

如果成熟 Memory / Context 系统 + 很薄 adapter 已经解决绝大多数问题，应缩小甚至放弃独立层。

---

## 11. 当前未冻结的关键问题

详见 [`open-questions.md`](./open-questions.md)。

最重要的包括：

- exact DB / schema；
- State、Claim、Skill 的最小必要模型；
- 生命周期和过期机制；
- Proposal 分级治理；
- Context Compiler 的 retrieval / rerank / budget 设计；
- Runtime Compiler 是否必要；
- provider contract；
- UI 信息架构；
- 外部数据接入；
- cross-device；
- privacy / disclosure granularity；
- benchmark 与真实长期 dogfood。

---

## 12. 文档职责

- [`origin.md`](./origin.md)：为什么这个项目出现；不要倒灌后来的架构。
- [`vision.md`](./vision.md)：希望最终形成怎样的用户体验和长期能力。
- [`design-state.md`](./design-state.md)：**当前 canonical design state**。
- [`principles.md`](./principles.md)：稳定原则。
- [`architecture.md`](./architecture.md)：当前实现基线。
- [`related-work.md`](./related-work.md)：现有系统解决了什么、我们不应重复声称什么。
- [`open-questions.md`](./open-questions.md)：尚未冻结的问题。
- [`design-notes/`](./design-notes/)：设计演化与关键讨论的历史记录。

后续发生冲突时，应优先判断：

1. 是不是 `origin` 与当前架构职责不同；
2. 是 Stable principle 被新证据推翻；
3. 还是仅仅某个 Baseline / Hypothesis 发生了变化。

不要因为一个新的实现想法就重新定义整个项目。
