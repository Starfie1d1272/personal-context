# Contributing

## 项目阶段

Personal Context 目前处于**早期设计与验证阶段**。仓库主要记录设计状态、研究假设和开放问题，尚未进入完整实现。

因此贡献的重点不是“实现新功能”，而是帮助当前设计变得清晰、可验证，并避免过早锁定没有依据的架构。

## 修改设计前先阅读

如果你要讨论或修改设计，请先阅读：

- [`docs/design-state.md`](docs/design-state.md) — 当前权威设计状态
- [`docs/principles.md`](docs/principles.md) — 已确定的设计原则
- [`docs/architecture.md`](docs/architecture.md) — 当前架构
- [`docs/open-questions.md`](docs/open-questions.md) — 尚未解决的问题

`docs/origin.md` 记录项目为什么出现，不应被后来的架构讨论改写。

## 新设计讨论的定位

新设计讨论应明确属于以下类型之一：

- 已确定原则的强化或修订；
- 当前方案调整；
- 新的待验证假设；
- 新的待解决问题。

不要因为一个新实现或新项目重新定义整个 Personal Context。

## 贡献原则

- 优先复用成熟基础设施，不重复设计已经存在的 Memory、RAG、Agent Runtime 或 Skills 运行时。
- PR 应小而聚焦，并说明修改了什么设计判断。
- 对当前设计语义的修改需要在 PR 中明确标注：是原则修订、方案调整，还是新增待验证假设。
- 不提前引入尚未存在的代码测试、release、CI 或版本管理流程。

## 中文文档语言风格

中文设计文档以自然中文为主。能够准确、自然表达的通用技术概念使用中文；Agent、Skill、项目名、协议名、代码/API 标识以及中文翻译会造成歧义的固定术语保留英文。英文原词仅在确有检索或消歧价值时首次括注。
