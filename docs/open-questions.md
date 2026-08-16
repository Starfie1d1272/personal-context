# Open Questions

> **As of:** 2026-08-16  
> **Status:** Intentionally unresolved  
>
> 这里记录尚未冻结的问题。它们不应被 README 或架构图误写成已经决定的事实。

---

## 1. Database / storage

### Q1.1 使用什么数据库？

当前只锁定：

> 核心 control plane 必须结构化、可查询、可追溯。

未决定：

- SQLite；
- PostgreSQL；
- embedded relational DB；
- graph DB；
- relational + graph hybrid；
- event-sourced / append-only + materialized view。

第一 prototype 很可能适合 SQLite，但这不是长期决议。

### Q1.2 State history 怎么表示？

候选：

- current row + revision table；
- temporal rows；
- event sourcing；
- provider-native temporal graph。

需要支持：

- current；
- historical；
- superseded；
- retired；
- revalidation。

### Q1.3 Claim 是否需要图结构？

Claim 至少需要：

```text
supports
contradicts
qualifies
scope
source
```

但是否必须使用 Graph DB 尚不明确。

### Q1.4 Skill 数据应该如何拆？

需要决定：

- structured metadata；
- Markdown instructions；
- executable code；
- evaluation history；
- versioning；
- provider ownership。

---

## 2. Ontology / semantics

### Q2.1 State / Claim / Skill 是否真的足够且必要？

这是设计 thesis，不是教条。

需要寻找：

- 无法自然归入三类的重要长期资产；
- 三类其实可被更简单模型替代的证据；
- 是否需要 Preference、Goal、Constraint 等独立一级概念。

### Q2.2 Preference 属于 State 还是独立类型？

例如：

```text
preferred package manager = pnpm
```

可能是：

- current State；
- durable preference；
- Skill precondition / policy。

需要明确语义。

### Q2.3 Goal / Task / Activity 的关系

当前倾向：

- Activity 是顶层触发；
- Task 只是 Activity 类型之一；
- Goal 可以是 durable state / objective。

仍需与 DSH 原生 Goal / Todo / Schedule 对齐。

---

## 3. Evidence and authority

### Q3.1 如何表示 authority？

需要避免简单粗暴的固定分数。

可能依赖：

- source type；
- subject；
- field；
- freshness；
- directness；
- user confirmation；
- tool verification；
- consistency。

### Q3.2 同一 Evidence 能支持多个资产吗？

例如一次 tool result 可能同时：

- 更新 State；
- 支持 Claim；
- 验证 Skill。

数据模型需要避免复制和丢失 lineage。

### Q3.3 DSH Session 之外的 Evidence 如何长期引用？

需要设计稳定 locator：

- file version；
- URL snapshot；
- Git commit；
- paper page；
- external API record；
- device probe。

---

## 4. Assimilation / write-back

### Q4.1 如何从新 Evidence 识别真正值得长期沉淀的变化？

这是可能比 retrieval 更困难的问题。

需要区分：

- transient observation；
- current State；
- durable Claim；
- Experience；
- Skill candidate；
- noise。

### Q4.2 Proposal 什么时候合并、什么时候并存？

尤其是：

- State conflict；
- Claim contradiction；
- duplicate Skill；
- evidence-only updates。

### Q4.3 如何防止 LLM 推测逐渐“洗白”为事实？

需要明确：

- provenance preservation；
- inference level；
- confirmation；
- authority inheritance rules。

---

## 5. Governance friction

### Q5.1 哪些更新可以自动提交？

当前候选：

```text
auto-commit
silent proposal
batch review
immediate approval
never infer
```

但没有证明这是正确分级。

### Q5.2 用户一天能接受多少治理动作？

这是产品生死问题。

需要长期 dogfood 测：

- proposal/day；
- review time；
- ignored proposals；
- false positive；
- wrong auto-commit；
- trust recovery。

### Q5.3 是否需要“观察一段时间再确认”？

某些偏好和状态可能不适合立即写入，而应：

```text
silent observations
→ accumulated evidence
→ proposal
```

---

## 6. Freshness / expiry / revalidation

### Q6.1 什么应该主动过期？

候选：

- device software version；
- location；
- project phase；
- active branch；
- dependency version。

有些信息可以 TTL，有些必须事件驱动。

### Q6.2 如何提示 stale？

可能需要：

```text
fresh
aging
stale
needs_revalidation
unknown
retired
```

### Q6.3 谁负责 revalidation？

- system probe；
- background schedule；
- next relevant Activity；
- user review；
- provider refresh。

需要避免重新走向高成本 autonomous heartbeat。

---

## 7. Context Compiler

### Q7.1 具体 retrieval stack 是什么？

第一版候选：

```text
structured query
+ SQLite FTS
+ embedding
+ reranker
```

或直接复用 OpenViking / other provider。

需要 benchmark，而不是靠直觉。

### Q7.2 Type-aware handling 真的能显著优于 mature retrieval 吗？

这是核心研究问题之一。

需要比较：

- BM25；
- vector；
- hybrid；
- reranker；
- MemOS；
- OpenViking；
- type-aware compiler。

### Q7.3 哪些 State 应该强制进入 Context？

如果当前 Activity 已明确关联项目 / 设备，一些 current State 可能不应该参与普通 similarity ranking。

需要 formalize。

### Q7.4 Claim 冲突如何编译？

如果两个重要 Claim 冲突：

- 都给？
- 给 summary？
- 按 scope？
- 按 evidence strength？
- 只在需要时展开？

### Q7.5 Context budget 如何动态分配？

需要考虑：

- State；
- Claim；
- Skill；
- Evidence；
- task material；
- model context window；
- latency / cost。

---

## 8. Runtime Compiler

### Q8.1 是否真的需要独立 Runtime Compiler？

可能的能力：

- Tool exposure；
- Skill exposure；
- preset composition；
- runtime defaults；
- provider choice。

但也可能通过 DSH Preset + Skill registry + Context prompt 足够解决。

必须 benchmark。

### Q8.2 能否让系统承担部分“模型智能”？

核心实验：

> 强模型第一次完成并沉淀后，第二次能否让明显更弱、更便宜的模型，以更少 token 和 tool calls 稳定完成？

如果不能，Runtime Compiler 的价值需要重新评估。

---

## 9. Disclosure / privacy

### Q9.1 Policy 粒度多细？

候选：

- asset；
- entity；
- field；
- scope；
- target model；
- provider；
- purpose；
- session；
- local / cloud。

### Q9.2 是否需要 redaction / generalization？

例如：

```text
local model
→ exact health value

trusted cloud
→ generalized range

public / unknown
→ deny
```

需要明确 policy language。

### Q9.3 credentials 是否永远 never-model？

可能需要独立 secret plane，而不是 Personal Context 普通 asset。

---

## 10. Provider federation

### Q10.1 Provider contract 应该多薄？

Memory Provider 可能只需要：

```text
search
recall
remember
revise
retire
```

但 State / Claim / Graph / Documents 不一定适合同一个接口。

### Q10.2 Provider 结果如何保留 provenance？

不能在聚合后只剩文本。

### Q10.3 如何禁止多个 Provider 各自直接注入？

需要 DSH 层面的 composition / policy 约定，或统一 pre-step owner。

---

## 11. External apps / cross-device

### Q11.1 大量闭源 App 无法连接怎么办？

不能假设：

```text
所有 Source of Truth
→ 一个 Adapter
→ 问题解决
```

需要研究：

- import/export；
- user-provided snapshots；
- accessibility / automation；
- local bridge；
- mobile companion；
- manual confirmation。

### Q11.2 多设备如何同步？

还没有设计：

- local-first sync；
- server；
- CRDT；
- encryption；
- conflict resolution；
- offline mode。

第一阶段可以不解决。

### Q11.3 是否应该让 Personal Context 自己成为某些领域的 Source of Truth？

例如小型 device / environment data 可以。

但财务、文献、Git 等应尽量保留专业系统。

边界需按领域决定。

---

## 12. Human UI

### Q12.1 第一版最小 UI 是什么？

当前候选：

```text
Overview
Entities
Knowledge
Skills
Activity
Sources
```

但 prototype 可能只需要：

- State table；
- Evidence trace；
- Proposal queue；
- Context Run；
- edit / accept / reject。

### Q12.2 怎样把治理做到低摩擦？

UI 不应该只是 Memory admin console。

需要研究：

- stale review；
- batch proposal；
- change diff；
- confidence / source display；
- one-click verify；
- context debugging。

---

## 13. DSH / project packaging

### Q13.1 一个 repo 还是 Core + Distribution 两个 repo？

当前默认：

- `personal-context` 保存核心；
- Personal DSH Distribution 以后按需要单独出现。

尚未需要立刻拆仓库。

### Q13.2 Core 如何保持 runtime-neutral？

需要防止 schema / API 直接依赖 DSH event internals。

### Q13.3 DSH developer preview 带来的兼容风险怎么控制？

第一版应尽量：

- 少量核心插件；
- 清晰 adapter；
- 不 fork / patch core；
- 避免十几个深耦合 package。

---

## 14. Evaluation

### Q14.1 最好的 benchmark 是什么？

不仅测 recall。

候选指标：

- required-context recall；
- irrelevant-context ratio；
- stale State rate；
- conflict coverage；
- sensitive disclosure rate；
- context token；
- latency；
- wrong write rate；
- provenance coverage；
- repeat-task tool calls；
- weaker-model success；
- user maintenance time。

### Q14.2 需要 synthetic benchmark 还是 long-term dogfood？

两者都需要：

- synthetic / curated context set：可控比较；
- dogfood：验证治理成本和长期状态正确性。

### Q14.3 什么结果意味着项目应该缩小？

如果成熟系统 + thin adapter：

- stale state 很少；
- disclosure 可控；
- user maintenance 更低；
- repeat-task efficiency 相当；

那么独立 Personal Context layer 的复杂度可能不值得。

---

## 15. Product / communication

### Q15.1 最终最强宣传点是什么？

尚未冻结。

目前相对有潜力的方向：

- Memory stays current, not just grows；
- structured state instead of model-managed Markdown；
- from retrieval to prepared agent environment；
- visible state / stale / proposals / context runs；
- system intelligence reduces model intelligence requirements。

不应再把“跨 AI”“Memory Viewer”“用户拥有 Memory”单独当成独有卖点。

### Q15.2 `Personal Context` 是否最终是产品名？

当前 repo 名继续保留。

未来如果出现 `Personal DSH Distribution`，它更可能是 delivery / distribution 名称，而不是替代核心概念。
