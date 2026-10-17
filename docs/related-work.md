# 相关工作与复用策略

核对日期：2026-10-17。DSH 与 MemOS 的结论固定于下列提交，依据官方文档、配置和源码；本轮未安装或运行该组合。其他项目保留为历史参考，具体能力与版本待采用时重新核查。

Personal Context 通过组合已有能力，形成由用户掌控的个人信息与长期认知系统。已有实现帮助确定复用范围；系统设计继续关注记录如何进入日常使用、如何随现实变化更新，以及如何为不同 Agent 准备合适的信息与能力。完整构想和逐步实践可以同时推进。

## 1. 按能力选择组成部分

| 能力 | 候选来源 | 本项目中的复用位置 | 需要回答的适配问题 |
| --- | --- | --- | --- |
| Agent 运行与能力组合 | DSH | 会话、工具、工作环境、Profile / Bundle 与界面扩展 | 版本是否兼容；当前活动如何映射到配置与权限 |
| 记忆捕获、检索与归纳 | MemOS | 经验、世界认识、Skills 和记忆查看 | 个人对象能否通过正式接口表达；默认模式启用了哪些能力 |
| 经验组织与上下文密度 | GA 等历史参考 | 索引、按需加载与 Skill 内容组织 | 哪些思路可由现有组件承载；如何处理漏取和误用 |
| 专业领域记录 | Git、Zotero、资产等应用 | 原始记录、稳定引用与必要的本地补充 | 谁维护原始值；来源变化如何影响派生认识 |
| 分层检索、时态关系 | OpenViking、Graphiti 等历史参考 | 具体场景确有需要时选用 | 现有组件是否足够；额外基础设施是否值得维护 |

表中的候选不构成必须同时引入的依赖清单。先用一条完整用户流程检查已有能力，再补充其中缺少的连接或领域操作。

## 2. DSH：首个工作环境

核对快照：[deepseek-ai/deepseek-harness @ 687f282](https://github.com/deepseek-ai/deepseek-harness/tree/687f282e8625c6b8d11ff58211a90524be2e4317)。该快照的 [agent package](https://github.com/deepseek-ai/deepseek-harness/blob/687f282e8625c6b8d11ff58211a90524be2e4317/packages/core/agent/package.json) 版本为 **0.2.1-rc.2**。

DSH 提供 Agent 运行和插件扩展的基础设施，可通过 Profile / Bundle 组合能力。会话事件、工具与 `agent/pre-step` 等接入点，为记录活动、按需准备上下文提供了落点；具体接入必须遵守对应事件和生命周期契约。参见 [DSH 架构](https://github.com/deepseek-ai/deepseek-harness/blob/687f282e8625c6b8d11ff58211a90524be2e4317/docs/architecture.md)。

Personal Context 可以先提供活动配置，连接记忆组件与领域应用，再按实际需要增加个人记录的查看、修正和上下文选择。会话日志可以作为来源之一，原始专业数据仍由其负责的应用维护。

### storage-domain 的适用范围

[storage-domain](https://github.com/deepseek-ai/deepseek-harness/blob/687f282e8625c6b8d11ff58211a90524be2e4317/packages/storage/storage-domain/README.md) 提供经过 schema 校验的 KV 领域记录，写入持久化后发出进程内变化事件。它可以承载少量自定义记录，但该快照明确存在以下限制：

- 无跨表事务、二级索引和多段键。
- 无自动数据迁移；存储版本与 schema 版本不一致时拒绝打开，需要自行迁移数据。
- 变化通知仅在当前进程可见。
- 存储层本身不向模型暴露工具或上下文，使用方需要提供相应入口。

因此，是否用它维护某类长期记录，应结合查询、迁移和使用方式决定。

## 3. MemOS：结构化记忆与现成 DSH 接入

核对快照：[MemTensor/MemOS @ a7367d0](https://github.com/MemTensor/MemOS/tree/a7367d07e55db61099f7b4e2c1108bc5831a24f3)。以下结论针对该仓库的 **local plugin**。

### 已有能力与复用位置

MemOS local plugin 已使用 SQLite 存储 traces、policies、world model、skills、episodes、feedback 和 audit 等内容。[存储说明](https://github.com/MemTensor/MemOS/blob/a7367d07e55db61099f7b4e2c1108bc5831a24f3/apps/memos-local-plugin/core/storage/README.md) 与[初始迁移](https://github.com/MemTensor/MemOS/blob/a7367d07e55db61099f7b4e2c1108bc5831a24f3/apps/memos-local-plugin/core/storage/migrations/001-initial.sql) 展示了 STRICT 表、字段约束、索引、外键和 JSON 内容，以及来源、归属、时间与生命周期字段。

结构化存储已经是可复用的基础。Personal Context 需要进一步映射个人领域语义，例如一次验光与当前镜片参数、购买与当前拥有、论文实验观察与作者主张。已有字段和正式接口能表达的直接采用；具体缺口再通过扩展或少量领域记录补足。长期个人数据的维护应优先依赖正式接口，避免直接修改上游内部数据库结构。

MemOS 的 [DSH adapter](https://github.com/MemTensor/MemOS/blob/a7367d07e55db61099f7b4e2c1108bc5831a24f3/apps/memos-local-plugin/adapters/deepseek-harness/README.md) 已提供树外 bundle：对接受的直接用户轮次执行有界自动召回，通过后台队列捕获结果，并提供六个模型工具和现有 Viewer。这些能力可作为整合起点。

### 默认行为与适配边界

| 核对事项 | 固定快照中的行为 | 对整合的影响 |
| --- | --- | --- |
| 默认记忆模式 | 默认启用 lightweight memory；L2 policy、L3 world model 和 skill crystallization 需显式切换到 full-memory 模式 | 分别记录组件具备的能力与实际启用的能力 |
| 模型工具 | 六个工具覆盖搜索、详情、时间线、环境和技能查询；技能读取还会记录使用或试用 | 个人对象的通用创建、修改和纠错入口需另查正式接口；这六个工具不能代表全部写入能力 |
| 查看界面 | Viewer 为独立页面，与 adapter 共用进程内 MemoryCore | 可以先使用现有界面，之后按需求增加 DSH 中的入口 |
| 写入时序 | 捕获在后台队列进行，下一轮召回读取当时已提交的状态 | 刚发生的变化可能尚未被下一轮取得，需要明确更新与可见性的关系 |
| 数据流向 | 数据库在本地；辅助 LLM 默认可委托给 DSH 当前模型路由，召回内容也进入模型提示 | 是否离开设备取决于提供方配置，披露范围需要覆盖这些调用 |

以上行为均来自该快照的 adapter README。配置和运行条件变化后，应重新核查。

### 版本兼容性尚待处理

[MemOS package.json](https://github.com/MemTensor/MemOS/blob/a7367d07e55db61099f7b4e2c1108bc5831a24f3/apps/memos-local-plugin/package.json) 声明 DSH peer 范围为 **>=0.1.0-rc.5 <0.2.0**，开发依赖为 **0.1.0-rc.6**。上述 DSH 快照的 **0.2.1-rc.2** 超出该范围。[DSH plugin-manager](https://github.com/deepseek-ai/deepseek-harness/blob/687f282e8625c6b8d11ff58211a90524be2e4317/packages/boot/plugin-manager/README.md) 会检查版本兼容性。两个固定快照不能据此描述为可直接安装使用的组合。

源码中还有一个具体适配线索：DSH 的 `agent/pre-step` 包装监听器需要保留 `startsRequestSeries`；MemOS [bridge.ts](https://github.com/MemTensor/MemOS/blob/a7367d07e55db61099f7b4e2c1108bc5831a24f3/apps/memos-local-plugin/adapters/deepseek-harness/bridge.ts#L268-L274) 在成功注入召回内容时重新构造 `kind` 与 `messages`，没有传递该标记。这是对两个快照的静态比较，尚未运行验证其影响，也不能据此判断全部兼容工作只涉及这一处。

实际探索应选择明确支持的版本组合，或完成适配后验证。放宽版本检查本身不能证明兼容。

## 4. 历史参考与待核查方向

下表保留 2026 年 8 月讨论中的参考范围，帮助追溯设计来源。本轮没有重新核实这些项目的身份链接、版本与能力；这里记录的是当时关注的问题，不能用作当前功能比较或成熟度判断。具体比较见历史记录 [2026-08-16 收敛笔记](design-notes/2026-08-16-convergence.md)，采用前需补官方来源与固定版本。

| 历史参考 | 当时关注的问题 | 再次采用前要核查什么 |
| --- | --- | --- |
| GenericAgent（GA） | 上下文信息密度、分层索引、经验与 Skill 的组织 | 按需加载机制、检索控制与使用失败后的修正方式 |
| OpenViking | 资源、记忆与 Skill 的层次组织和检索 | 可复用接口、部署成本及与现有组件的重叠 |
| Graphiti | 时态关系、来源与当前 / 历史认识 | 时间语义、修订和删除机制是否符合具体记录 |
| TencentDB Agent Memory | 记忆的责任方、访问范围与 Agent 装配 | 当前项目身份、部署边界、权限及数据流向 |
| PowerContext、Mnemon | 持久上下文、记忆空间与使用界面 | 接口、维护状态与现有运行时的职责划分 |
| dsh-native-memory、dsh-memory-evolve | DSH 生态中的记忆持久化与更新方式 | 对应版本、插件契约、来源管理与导出方式 |

## 5. 整合系统的价值与下一步

Personal Context 关注一条连续的用户流程：信息进入系统后可直接查看和纠正，现实变化更新当前认识，Agent 按活动取得合适的上下文与能力，使用反馈再影响后续记录与选择。不同组件可以共同完成这条流程，自己的补充范围由实际缺口决定。

下一步可以选设备或物品记录这一类简单场景，先对照现有接口检查来源、状态更新和按需使用是否连贯。确定兼容组合后，再用合成数据做小型接入，记录真正跑通的部分与待处理问题。系统构想、配置示例和实际运行结果分别陈述；公开设计文档不以完整产品或对比实验为前提。

实现中的数据和读写机制见 [架构设计](architecture.md)，当前选择见 [设计状态](design-state.md)。
