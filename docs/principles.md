# Design Principles

> **As of:** 2026-08-16  
> **Status:** Stable unless explicitly revised  
>
> 本文件只保存当前认为应当长期稳定的设计原则。实现细节、Provider 选择和具体 schema 不写在这里。

---

## P1. 用户拥有的是长期 Context，不是某个模型的附属 Memory

关于用户、项目及其现实环境的长期认识，其生命周期天然长于一次会话、一个模型或一个 Agent。

模型、Agent 和 Memory Provider 都应被视为消费者或维护者，而不是天然的数据所有者。

这是一条愿景原则，不应被宣传成“单点技术首创”。

---

## P2. 核心控制面必须结构化

重要长期状态不能主要依赖：

```text
Markdown + LLM 自己阅读 + LLM 自己理解 + LLM 自己改写
```

核心 control plane 必须：

- schema-backed；
- queryable；
- tool-mediated；
- versionable；
- traceable；
- policy-enforceable。

但长文本、SOP、Episode、Skill docs 和原始资料可以继续使用 Markdown / files / blobs。

> **控制面结构化，内容面自由。**

---

## P3. Memory 是机制，不是统一语义

Memory 可以保存和召回 State、Claim、Experience、Skill。

但长期信息不能因为进入 Memory 系统就失去其原本的决策语义。

---

## P4. State / Claim / Skill 的治理语义不同

```text
State
→ 现在是什么？

Claim
→ 我们有什么理由相信什么？

Skill
→ 在什么条件下应该怎么做？
```

它们的冲突、更新、失效和验证方式不同，因此不能默认使用同一个“memory overwrite”逻辑。

---

## P5. Evidence 不等于 Truth

任何长期资产都应能回答：

> **“凭什么？”**

用户一句话、Agent 总结、Tool result、文件、论文、系统探针都只是不同 authority 的 Evidence。

Evidence 进入长期认知前需要解释和治理。

---

## P6. Agent 可以提出修改，但不天然拥有长期写入权

默认写入协议：

```text
Evidence
→ Proposal
→ authority / conflict / risk / policy
→ Governance
→ Commit
```

低风险、高权威来源未来可以自动提交；高风险或高敏感信息可能永远不允许自动推断。

具体分级仍属于实现问题。

---

## P7. 生命周期是一等属性

长期系统不能只有：

```text
remember()
```

还必须考虑：

```text
verify()
revise()
supersede()
expire()
revalidate()
retire()
```

旧信息不能因为“曾经被记住”就永久影响当前 Agent。

---

## P8. Context 的目标不是更多，而是正确、相关和最小必要

长期信息不能全量进入每一轮 Prompt。

Context selection 的目标包括：

- current；
- relevant；
- authoritative；
- minimal；
- within budget；
- allowed to disclose。

---

## P9. 相关性可以概率化，权力边界必须确定

可以使用：

- BM25；
- embedding；
- reranker；
- graph retrieval；
- temporal retrieval；
- classifier；
- 小 LLM。

来判断相关性。

但以下边界不能交给主模型临场“凭感觉”：

- identity / scope；
- current authoritative state；
- revoked / expired；
- privacy；
- disclosure；
- allowed provider / model；
- token hard limits；
- serialization constraints。

---

## P10. Provider 可以很多，最终 Assembly / Disclosure Owner 必须唯一

不同 Provider 可以返回候选。

但一次 Agent request 必须有一个最终责任方决定：

> **模型最后真正看到了什么？**

否则无法审计 Context 的来源、重复、冲突和泄露。

---

## P11. 所有进入长期认知范围的内容都必须可见

Personal Context 不能成为新的黑箱 Memory。

至少要能够：

- 看；
- 搜；
- 改；
- 删；
- 导出；
- 追溯来源；
- 查看历史；
- 查看冲突；
- 审核 Proposal；
- 查看披露规则；
- 查看每次 Context Run。

系统必须能回答：

> **“你为什么知道这件事？”**

> **“为什么这个 Agent 能看到它？”**

---

## P12. DSH 是第一 Runtime，不是核心数据所有者

第一实现应尽可能复用 DSH 的基础设施。

但核心语义和数据库必须能够脱离 DSH 被其它 Runtime / MCP / Agent 使用。

---

## P13. Event-driven 优先

系统应在有实际触发时工作：

- input；
- event；
- change；
- explicit schedule；
- revalidation requirement。

不把持续 autonomous heartbeat 作为默认架构。

---

## P14. 稳定核心语义，开放领域扩展

不要提前构建“全世界 ontology”。

核心保持小而稳定；设备、科研、健康、资产、财务等通过领域 schema / packs 扩展。

---

## P15. 外部集成是增量能力，不是成立前提

不假设所有闭源 App 都可连接。

优先从系统可靠可观察的 Source 获得价值，再逐步增加 Adapter。

---

## P16. 系统必须允许自己被证明“不值得存在”

Personal Context 是设计与研究假设，不是先验正确答案。

如果：

```text
DSH + MemOS / OpenViking / other mature systems
+ thin adapter
```

已经能以更低复杂度实现相同结果，则应缩小项目，而不是为了维护项目身份继续造层。
