# Architecture

> **As of:** 2026-08-16  
> **Status:** Current baseline, not final specification  
>
> Stable principles are in [`principles.md`](./principles.md).  
> Unresolved choices are in [`open-questions.md`](./open-questions.md).

---

## 1. System boundary

Personal Context 不重新实现：

- Agent Runtime；
- LLM provider system；
- full Memory OS；
- full RAG stack；
- Skills Runtime；
- Git；
- Zotero；
- 财务软件；
- 家庭资产软件；
- universal ontology。

核心负责的是：

```text
Personal Data Model
+ Provenance & Authority
+ Lifecycle
+ Update Proposals
+ Context Assembly
+ Disclosure Policy
+ Human Governance UI
```

---

## 2. Top-level architecture

```text
┌──────────────────────────────────────────────────────────┐
│                     Evidence Sources                     │
│                                                          │
│ User / DSH Session / Tool results / Git / Files / Probe │
│ Papers / APIs / Memory providers / External adapters    │
└───────────────────────────┬──────────────────────────────┘
                            │
                            ▼
                       Evidence Layer
                            │
                            ▼
              Candidate interpretation / Proposal
                            │
                  authority / conflict / risk
                            │
                            ▼
                       Governance
                            │
                            ▼
┌──────────────────────────────────────────────────────────┐
│                Personal Context Control Plane            │
│                                                          │
│ Entity / State / Claim / Skill metadata / EvidenceRef   │
│ Proposal / Relation / Policy / Scope / Revision         │
│ Lifecycle / Disclosure / Context Run                    │
└───────────────────────┬──────────────────────────────────┘
                        │
                        ▼
                 Context Assembly
           scope / retrieval / rerank / types
             budget / disclosure / serialize
                        │
          ┌─────────────┴──────────────┐
          ▼                            ▼
   Context Compiler              Runtime Compiler?
  what model sees            tools / skills / defaults
          │                            │
          └─────────────┬──────────────┘
                        ▼
                       DSH
                 reasoning / action
                        │
                        ▼
                    new Evidence
```

`Runtime Compiler` 目前仍是 Hypothesis；`Context Compiler` 是 Baseline。

---

## 3. Structured control plane

### 3.1 Logical entities

当前建议的最小逻辑对象：

```text
Entity
State
Claim
Skill
EvidenceRef
Proposal
Relation
Policy
Scope
Revision
ContextRun
```

这不是最终数据库 schema。

### 3.2 State

State 表达：

> **当前认为“现在是什么”。**

可能需要的元数据：

```text
subject / entity
key
value
valid_from
valid_until
observed_at
verified_at
source / evidence refs
authority
confidence
freshness
sensitivity
status
supersedes / superseded_by
```

State 重点是 currentness 与 authority。

### 3.3 Claim

Claim 表达：

> **某个主体提出或支持的可论证判断。**

需要允许：

```text
support
contradict
qualify
scope
confidence
source attribution
```

Claim 不采用 State 的“新值覆盖旧值”语义。

### 3.4 Skill

Skill 可以拆成：

```text
structured metadata
├── preconditions
├── applicability
├── reliability
├── verification criteria
├── history
└── lifecycle

content / executable
├── Markdown instructions
└── code / tool / workflow
```

### 3.5 EvidenceRef

Personal Context 不必复制所有原始数据。

EvidenceRef 应能够指向：

- DSH session + seq；
- tool call/result；
- Git commit；
- file + range；
- URL；
- paper / page / figure；
- API record；
- external provider object。

---

## 4. Write path: assimilation

### 4.1 Default pipeline

```text
new event
  ↓
Evidence capture
  ↓
candidate extraction
  ↓
asset classification
  ↓
authority evaluation
  ↓
conflict detection
  ↓
Proposal
  ↓
governance
  ↓
commit / reject / revise
```

### 4.2 Why not direct model writes

例如：

```text
用户：最近 Codex 好像老断。
```

首先应该得到：

```text
Evidence:
user reported repeated disconnects
```

而不是直接得到：

```text
State:
Codex connectivity = broken
```

更不能直接得到：

```text
Claim:
Japanese nodes are unreliable
```

这三者需要不同的证据门槛。

### 4.3 Authority

不同 source 应有不同 authority。

例如：

```text
git remote -v
```

对于 repository.remote 的 authority 通常高于一次模型总结。

具体 authority model 仍未冻结。

### 4.4 Governance levels

候选模型：

```text
auto-commit
silent proposal
batch review
immediate approval
never infer
```

当前只确认“必须分级”，不确认最终五级设计。

---

## 5. Read path: Context Assembly

### 5.1 Activity

顶层触发不是只有 Task。

可能包括：

- task；
- query；
- record；
- conversation；
- event；
- schedule；
- system probe。

Activity 决定是否：

- 只记录 Evidence；
- 查询 State；
- 召回 Memory；
- 加载 Skill；
- 运行 Agent；
- 生成 Proposal。

### 5.2 Hard scope

优先使用确定信息缩小范围：

```text
current user
workspace
project
session
agent
preset
model
tool
explicit entity
domain
privacy scope
```

### 5.3 Candidate retrieval

可以并行使用成熟组件：

```text
exact structured query
FTS / BM25
vector retrieval
graph query
temporal query
memory provider
document provider
skill provider
```

Personal Context 不应发明自己的全套 retrieval stack。

### 5.4 Fusion / rerank

候选可以使用：

- RRF；
- weighted fusion；
- dedicated reranker；
- small classifier。

这些是基础设施，不是项目创新重点。

### 5.5 Type-aware handling

不同资产进入最终 Context 前使用不同规则。

#### State

考虑：

- current；
- authority；
- validity；
- freshness。

某些当前 State 可能是强制 Context，而不是普通 similarity candidate。

#### Claim

考虑：

- scope；
- confidence；
- supporting evidence；
- contradiction。

如果存在关键冲突 Claim，不能只选择最高分一条造成选择性证据。

#### Skill

考虑：

- precondition match；
- environment compatibility；
- reliability；
- historical success / failure。

### 5.6 Context budget

Context 必须有显式预算。

可以分层：

```text
L0 pointer
L1 summary
L2 full content
```

需要时再展开。

### 5.7 Disclosure

最终 Disclosure 必须晚于相关性判断，并使用确定性规则：

```text
candidate
  ↓
identity / scope
  ↓
sensitivity
  ↓
target model / provider
  ↓
policy
  ↓
redaction / deny / allow
  ↓
serialization
```

Sensitive data leakage 不能依赖 reranker “大概率不选中”。

---

## 6. ContextRun audit

每次 Context Assembly 应产生可审计记录，例如：

```text
Agent Context Run

Activity:
  install project dependencies

Agent:
  DSH / code preset

Included:
  current project
  current OS
  package_manager = pnpm
  dependency-install Skill

Excluded:
  health/*
    reason: disclosure policy

  unrelated-project/*
    reason: irrelevant

Sources:
  Personal Context State
  DSH Session
  Skill Provider

Tokens:
  ...
```

这是 Human UI 和 debugging 的重要基础。

---

## 7. DSH integration

第一实现预计利用：

### Session Log / Session Query

作为主要 Evidence 来源之一。

Personal Context 保存：

```text
EvidenceRef:
  session_id
  seq
```

不复制一套聊天历史数据库。

### Storage / Storage Domain

可以作为第一实现的 structured persistence substrate。

但核心 domain semantics 不能和 DSH implementation 强绑定。

### `agent/pre-step`

适合作为 Context Assembly 的重要 integration point：

```text
Activity
→ Personal Context
→ compile
→ final context
→ model
```

### Profile / Bundle / Preset

可以组合：

- Personal Context；
- Memory Provider；
- Skills；
- GA-inspired evolution；
- domain packs；
- UI。

这是一种 delivery/composition 方式，不是核心创新。

---

## 8. Memory and learning providers

当前原则：

```text
Memory Provider
→ candidate memory / experience / learned policy

Personal Context
→ canonical semantics / governance / assembly
```

MemOS、OpenViking、Graphiti 等都可以成为 Provider 或 baseline。

Provider 的派生输出不能默认覆盖 canonical State。

---

## 9. GA-inspired Learning Loop

保留 GA 的两个重要思想：

- Context information density；
- Experience → reusable SOP / Skill。

但降低以下设计的权力：

- LLM 自己维护文本事实库；
- tiny Markdown index 作为 authoritative state discovery；
- 模型自己决定所有事实何时应该读取；
- 经验自动变成世界事实。

可能的独立组件：

```text
experience-evolver
skill-evolver
```

---

## 10. Runtime Compiler hypothesis

Context Compiler 解决：

> 模型应该看到什么？

Runtime Compiler 可能解决：

> Agent 应该拥有什么？

例如：

```text
current environment
→ choose tools
→ choose skills
→ preset modifications
→ runtime defaults
```

它的目标是让 Harness 承担部分原本依赖模型能力的工作。

需要用 benchmark 证明独立价值。

---

## 11. Human UI

当前候选信息架构：

```text
Overview
Entities
Knowledge
Skills
Activity
Sources
```

### Overview

- 当前关键 State；
- stale / needs-review；
- pending Proposal；
- recent changes。

### Entities

- 人；
- 项目；
- 设备；
- 资产；
- 应用；
- 研究主题；
- 其它 domain entity。

### Knowledge

- Claims；
- support / contradiction；
- scope；
- uncertainty。

### Skills

- preconditions；
- reliability；
- usage history；
- failure history；
- revisions。

### Activity

- input；
- Tool result；
- Evidence；
- Proposal；
- commits；
- Context Runs。

### Sources

- DSH；
- Git；
- Files；
- Zotero；
- Memory Providers；
- APIs；
- system probes。

---

## 12. External sources

不要假设所有 Source of Truth 都能直接连接。

第一阶段优先：

```text
DSH Session
Tools
Workspace
Files
Git
System probes
Explicit user input
```

开放 API / MCP / local-first applications 是后续高价值 Adapter。

闭源、跨设备、无 API 应用的接入策略仍是 Open Question。

---

## 13. First vertical slice

# Device & Project Environment

需要支持：

```text
device
os
arch
shell
project
package manager
runtime / dependency environment
```

验证：

```text
State
→ Context Assembly
→ Agent action
→ Tool Evidence
→ Proposal
→ Governance
→ State update
```

第一版不要求同时接完整 Memory OS。

---

## 14. Non-goals

当前第一阶段明确不做：

- 新 Agent Runtime；
- 新 vector DB；
- 完整 RAG；
- 通用时间知识图数据库；
- 新 Skills Runtime；
- 完整财务软件；
- 完整家庭资产软件；
- universal ontology；
- 全自动 Personal AI 管家；
- “把所有个人 App 都同步进来”。

---

## 15. Implementation questions

以下内容不在本文锁定，见 [`open-questions.md`](./open-questions.md)：

- DB technology；
- exact schema；
- migrations；
- graph vs relational；
- authority scoring；
- revalidation；
- provider API；
- Runtime Compiler；
- governance UX；
- cross-device；
- external app ingestion；
- privacy rule language；
- Context benchmark。
