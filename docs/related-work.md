# Related Work and Reuse Strategy

> **As of:** 2026-08-16  
> **Status:** Working comparative notes  
>
> 这份文档记录当前讨论中已经形成的生态判断，用来约束项目不要把已有能力重新包装成“创新”。  
> 在对外做具体功能比较或版本级声明前，应再次核对上游仓库的最新实现。

---

## 1. 基本原则

现有生态已经覆盖了大量基础问题。

当前不能把以下单点当成 Personal Context 的独有创新：

- long-term memory；
- cross-session recall；
- layered context；
- hybrid retrieval；
- embedding / reranker；
- temporal facts；
- provenance；
- ACL / permissions；
- Agent loadout；
- Experience → Skill；
- context injection；
- Memory Viewer；
- 多 Agent 共享某种 Memory。

Personal Context 的研究问题必须建立在这些已有能力之上。

---

## 2. DeepSeek Harness (DSH)

### 已解决 / 可复用

- Agent Runtime；
- plugin / capability seams；
- Session；
- model / tool abstraction；
- Skills；
- storage；
- permissions / approval；
- sandbox；
- workflow / jobs；
- client UI extension；
- Profile / Bundle / Preset；
- external protocols / subagents。

### 对本项目的意义

DSH 非常适合作为第一 Runtime 和 dogfood 环境。

尤其重要：

- Session Log 可以成为 Evidence Ledger；
- pre-step 是 Context Assembly 的实现落点；
- storage-domain 可以承载第一版 structured data；
- Profile / Bundle 可以承载 Personal DSH distribution。

### 不是我们的创新

- “everything is a plugin”；
- 可插拔 Agent framework；
- 在 model call 前注入 context；
- Agent permissions / sandbox。

---

## 3. GenericAgent (GA)

### 已解决 / 启发

GA 的重要思想包括：

- Contextual Information Density；
- 极小长期索引；
- layered Memory；
- Experience → SOP / code / Skill；
- self-evolution；
- 强调让有限 context 中保留最具决策价值的信息。

### 真实使用中暴露的缺口

GA 的极小文本索引依赖：

```text
LLM 看懂 hint
→ 意识到深层内容存在
→ 主动 file_read
→ 正确理解
→ 在决策中正确使用
```

强模型下很优雅，但可靠性仍依赖模型：

- 可能没意识到应该读；
- 可能读到了却没有正确使用；
- Markdown 被迫承担部分 retrieval policy / semantics；
- 文本事实库不适合作为 authoritative current state。

### 对本项目的取舍

保留：

- information density；
- Experience / Skill evolution；
- Markdown 作为 SOP / Skill / Episode 内容载体。

替换或降权：

- LLM-maintained authoritative text facts；
- 让 Markdown 承担 DB / ACL / state machine；
- 让主模型成为长期事实 retrieval controller。

---

## 4. MemOS

### 已解决 / 可复用

当前讨论中将 MemOS 视为成熟的：

- long-term Memory；
- recall / capture；
- experience / policy / world-model-like learning；
- Skill evolution；
- DSH integration；
- Viewer / management surface。

### 对本项目的意义

MemOS 是：

- 重要参考实现；
- Memory / Learning Provider 候选；
- 主要 baseline。

### 不能再宣传为差异化

- “用户能看到 Memory”；
- “不同 Agent 共享长期 Memory”；
- “Experience 变成 Skill”；
- “DSH 有长期记忆”。

### 仍需上层治理的问题

MemOS 或其它 Memory Provider 的派生认识进入 Personal Context 时，不能因为被模型抽取出来就自动成为 authoritative State。

默认应进入：

```text
Evidence / Candidate / Proposal
```

再由 Personal Context 决定是否沉淀。

---

## 5. OpenViking

### 已解决 / 可复用

讨论中将 OpenViking 视为重要 Context Database / retrieval baseline：

- Resource / Memory / Skill；
- hierarchical organization；
- L0 / L1 / L2；
- semantic / hierarchical retrieval；
- reranking；
- session extraction；
- retrieval trace。

### 对本项目的约束

以下不能单独声称创新：

- 分层 Context；
- “只取相关信息”；
- hierarchical retrieval；
- Context Database 本身。

### 可能的角色

- retrieval provider；
- context baseline；
- 下层 Content / Memory provider。

---

## 6. Graphiti

### 已解决 / 可复用

Graphiti 已经证明以下问题有成熟实现：

- temporal facts；
- valid / invalid time；
- provenance / episode linkage；
- graph relationships；
- current vs historical fact；
- hybrid retrieval。

### 对本项目的约束

不能把：

```text
valid_from
invalid_at
provenance
temporal fact graph
```

本身宣传成创新。

### 仍然不同的问题

Personal Context 更关心：

- authoritative current State 如何进入最终 Agent Context；
- Claim 与 State 的不同冲突语义；
- final disclosure / assembly；
- user governance。

Graphiti 可以成为 State / temporal provider，而不是被重写。

---

## 7. TencentDB Agent Memory

### 已解决 / 可复用

讨论中关注其：

- Chat Memory / Skill / Wiki / CodeGraph；
- Owner / ACL；
- Agent loadout；
- version / status；
- BM25 + vector + fusion。

### 对本项目的约束

不能把以下单点当成创新：

- Memory 有 ACL；
- 不同 Agent 看到不同知识；
- 权限过滤后再 retrieval；
- hybrid retrieval。

### 仍值得研究

Personal Context 进一步关心：

- field / record-level disclosure；
- target model / provider-specific disclosure；
- redaction；
- final Context responsibility；
- full ContextRun audit。

---

## 8. PowerContext / Mnemon / dsh-native-memory / dsh-memory-evolve

这些项目在此前讨论中主要作为 DSH 生态参考。

### PowerContext

值得关注：

- durable context；
- experience / skill；
- DSH adapter；
- context provider 角色。

适合作为 Provider 候选，而不是 canonical personal state。

### Mnemon

值得关注：

- runtime memory；
- documents；
- memory spaces；
- supervised writes；
- UI；
- cross-agent sharing。

与 Personal Context 接近，但重点仍然偏 Memory。

### dsh-native-memory

设计上值得参考：

- DSH storage-domain；
- session-query；
- approval-gated write；
- provenance；
- secret detection；
- deterministic recall；
- export。

### dsh-memory-evolve

覆盖面很广，包含 Memory / Skill / Todo / project log / multi-session / external AI 等。

当前更适合作为设计参考，而不是核心依赖，避免形成两个 orchestration owners。

---

## 9. 当前真正的差异化假设

经过排除后，项目可能有价值的不是一个新算法，而是以下组合：

```text
schema-backed long-term control plane
+
State / Claim / Skill decision semantics
+
Evidence → Proposal → Governance → Commit
+
freshness / expiry / revalidation
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

目前应称为：

> **architectural hypothesis / design thesis**

而不是首创声明。

---

## 10. 当前宣传上应避免的误区

不应把以下作为头部差异化：

### “Your context belongs to you”

这是重要愿景，但用户自主、local / self-hosted、可导出等概念并非空白市场。

### “多个 AI 共享同一份 Memory”

MemOS 等已有系统已经覆盖相当多。

### “用户可以查看 AI 记住了什么”

Memory Viewer 并不独有。

### “只给 Agent 相关 Context”

GA、OpenViking、MemOS、成熟 RAG 系统都已经深入研究。

### “时间 State / provenance”

Graphiti 等已经构成强对照。

更有辨识度的产品主张应围绕：

- Memory 会维护和过期，而不是只增长；
- 结构化长期状态而非模型维护 Markdown；
- Agent 不只是“搜到”，而是得到正确的 Context / capability；
- 用户能看到当前世界模型、stale items、Proposal 和 ContextRun；
- 系统能否把强模型一次探索沉淀成以后更便宜模型可稳定复用的能力。

---

## 11. Reuse first

第一阶段应优先复用：

| Capability | Preferred strategy |
|---|---|
| Agent Runtime | DSH |
| Session / Evidence log | DSH |
| Memory / learning | MemOS / other provider |
| hierarchical context | OpenViking or similar |
| temporal graph | Graphiti or similar |
| retrieval | existing FTS / vector / reranker |
| Skill runtime | DSH / existing ecosystem |
| domain applications | adapters where realistic |

本项目最不应该做的事情之一，就是为了看起来“原创”而重写成熟基础设施。
