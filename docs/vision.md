# Vision

> **As of:** 2026-08-16  
> `origin.md` 记录项目为什么出现；本文描述如果这个方向成立，最终希望用户获得怎样的能力。

---

## 1. North Star

长期使用 AI 后，用户不应该得到越来越多互相独立的“聊天记忆”，而应该逐渐形成一份：

- 关于自己和当前环境的认识；
- 关于项目和研究的当前状态；
- 有证据支持、允许存在冲突的长期判断；
- 已经验证过、可以复用的 Skills；
- 清楚知道来源、更新时间和适用范围的信息；
- 可以被不同 Agent 按当前活动选择性使用的 Context。

这份长期认识首先由用户拥有和治理。

---

## 2. 理想体验不是“AI 记得更多”

一个长期使用数月的 Personal Context 系统，应该让用户感受到：

### 它知道“现在”而不是只记得“过去”

例如：

- 当前电脑和系统环境；
- 当前项目依赖；
- 当前研究问题；
- 当前设备状态；
- 最近已经被替代的旧配置。

如果某条信息可能过期，系统应该显式知道它“不再可信”或“需要重新确认”。

### 它知道自己为什么这样认为

任何重要 State / Claim 都能追到：

- 用户原话；
- DSH Session；
- Tool result；
- Git；
- 文件；
- 论文；
- 系统探针；
- 外部数据源。

### 它会把新的工作沉淀回来

一次排查、研究、开发或真实世界变化结束后，不只是留下聊天记录。

系统可以从新的 Evidence 中提出：

- State update；
- Claim update；
- Skill candidate。

但这些变化不会被模型随意写入。

### 它会随着使用减少重复探索

如果过去已经验证过一种做法，下一次应该能：

- 更少 token；
- 更少 Tool calls；
- 更少试错；
- 甚至让更弱、更便宜的模型稳定完成。

### 它不会把“我的全部人生”塞给每个 Agent

每次 Activity 只生成最小必要 Context。

一个 coding Agent 不需要看到健康数据；一个旅行任务也不需要读取所有科研 Claim。

### 用户自己能够看见整个长期系统

不是一个只能给 Agent 调用的隐藏 Memory API。

用户应该有一个长期认知控制面板，可以直接查看：

- 现在系统认为我的世界是什么状态；
- 哪些内容可能已经过期；
- 最近产生了哪些 Proposal；
- 哪些 Claims 存在冲突；
- 哪些 Skills 最近成功或失败；
- 某条信息从哪里来；
- 哪个 Agent 能看到什么；
- 一次 Agent request 最终实际看到了什么。

---

## 3. 长期形态

理想架构更接近：

```text
             Human
               │
      browse / edit / govern
               │
       Personal Context
       ├── State
       ├── Claims
       ├── Skills
       ├── Evidence
       ├── Policies
       └── History
               │
      ┌────────┼────────┐
      ▼        ▼        ▼
    Apps      DSH    Other Agents
               │
        Context / Runtime
           compilation
```

Personal Context 不要求所有应用的数据都迁入同一个数据库。

它可以：

- 自己保存 canonical metadata；
- 引用外部 Evidence；
- 从开放 Source 同步；
- 对不能连接的系统只保持手工输入或导入。

---

## 4. DSH 的角色

第一阶段 DSH 是最现实的 primary runtime：

- everything-is-a-plugin；
- Profile / Bundle / Preset；
- Session；
- Tools；
- Skills；
- storage；
- approval；
- sandbox；
- UI extension；
- external protocol。

因此可以在 DSH 上 dogfood 整套系统。

但长期目标不是把用户数据锁回另一个 Agent Runtime。

---

## 5. 与 Memory / Learning 的关系

成熟 Memory 系统应该被复用，而不是重造。

长期希望形成：

```text
Canonical Reality
State / provenance / lifecycle
             │
             ├──────────┐
             │          │
             ▼          ▼
      Context use   Lifelong Learning
                    Experience → Skill
```

Memory 和 Skill learning 帮助系统成长；Canonical Context 防止“模型记住了一句话”自动变成长期事实。

---

## 6. 领域愿景

最初构想包括：

- 设备 / IT 环境；
- 项目；
- 科研；
- Todo；
- 资产；
- 财务；
- 健康；
- 家庭设备；
- 语音记录。

这些是愿景来源和潜在 Domain Packs，不代表核心仓库要重新实现所有垂直应用。

现实策略是：

1. 先把 Agent 自己能够可靠观察的环境做好；
2. 再接少量开放、可信的数据源；
3. 最后根据真实需求决定哪些领域值得专门适配。

---

## 7. 成功标准

项目真正成功不是因为：

- Memory recall 更高 3%；
- 多了一个漂亮 Viewer；
- 支持更多 embedding；
- 能连接更多 API。

而是几周、几个月长期使用后：

> 用户和 Agent 对“现在是什么、我们知道什么、应该怎么做”形成了更准确、连续、可追溯、可控制的共享认识，而且维护成本足够低。

如果不能做到这一点，愿景应收缩。
