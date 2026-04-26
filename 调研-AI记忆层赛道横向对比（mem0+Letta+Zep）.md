# 调研补充：AI 记忆层赛道横向对比

> 调研时间：2026-04-27
> 配合阅读：`调研-mem0 AI记忆层框架.md`
> 本文补充 Letta（原 MemGPT）和 Zep，三者一起构成这个赛道的主要玩家图景

---

## 一、Letta（原 MemGPT）

- **GitHub**：https://github.com/letta-ai/letta
- **定位**：有记忆、能自我改进的 AI Agent 框架

### 核心思路

Letta 来自学术论文 MemGPT（2023），原始想法是用「分页」的方式模拟操作系统内存管理——把有限的 context window 当作"内存"，把外部存储当作"磁盘"，Agent 自己决定什么时候把信息写入外部、什么时候调回来。

后来商业化后演变为完整的 Agent 平台，核心记忆机制改为 **memory_blocks**：

```
Agent 的记忆由若干个命名块构成：
- human 块：关于用户的信息（"叫什么、喜欢什么、做什么"）
- persona 块：Agent 自己的人格和知识
- 自定义块：可以添加任意业务数据
```

这些块都在 context window 里，Agent 可以直接读取，也可以主动调用工具把外部信息写进来或读出去。

### 现在主推什么

- **Letta Code**：本地 CLI Agent，类似 Claude Code，能帮你写代码、做任务，有长期记忆
- **Letta API**：给开发者用的 Agent API，创建有状态 Agent，发消息，Agent 记得之前的上下文
- **Skills & Subagents**：Agent 可以调用技能包，也可以生成子 Agent 协作

### 技术特点

- 完全模型无关（支持 Claude、GPT、本地模型等）
- Agent 记忆随对话自动更新，不需要手动触发
- 有托管云服务（app.letta.com）+ 可自托管

---

## 二、Zep

- **GitHub**：https://github.com/getzep/zep
- **定位**：面向生产环境的 Agent 上下文工程平台

### 核心思路

Zep 的关键词是**「时序知识图谱」**（Temporal Knowledge Graph），底层由他们开源的 **Graphiti** 框架驱动。

区别于其他方案，Zep 不只存「事实」，而是存**关系随时间的变化**：

- 每条知识都有 `valid_at` 和 `invalid_at` 时间戳
- 比如"Alice 住在北京"可以被"Alice 搬去上海了"正确更新，而不是简单覆盖
- Agent 能理解"现在的事实"和"历史上的事实"的区别

**运作流程：**
1. **喂入**：把聊天记录、业务数据、文档、应用事件都输入给 Zep
2. **自动建图**：Zep 提取实体和关系，维护知识图谱
3. **检索组装**：查询时返回预格式化的、关系感知的上下文块，直接注入 prompt

### 现状

- **社区版（开源）已停止维护**，代码移到 `legacy/` 目录
- 现在主推 **Zep Cloud**（托管服务），SOC2 Type 2 / HIPAA 合规，主打企业客户
- 检索延迟 <200ms，面向生产环境
- 底层 Graphiti 仍然开源（https://github.com/getzep/graphiti）

---

## 三、三者横向对比

| 维度 | mem0 | Letta | Zep |
|------|------|------|------|
| **核心技术** | 向量存储 + LLM 提取 | memory_blocks（in-context 块）+ 外部存储 | 时序知识图谱（Graphiti）|
| **记忆粒度** | 碎片化事实（"喜欢深色模式"）| 结构化块（human/persona/自定义）| 关系 + 时间线（带 valid_at）|
| **开源程度** | 完全开源（Apache 2.0）| 开源 | 核心已闭源，底层图谱开源 |
| **适用场景** | AI 个性化、通用记忆 | 有状态 Agent、自我改进型 Agent | 企业级生产 Agent、复杂业务上下文 |
| **检索方式** | 语义 + BM25 + 实体融合 | in-context 直接读取 | 图谱关系查询 + 语义 |
| **更新机制** | 只 ADD，永不覆盖 | Agent 主动写入 | 时间线更新，保留历史 |
| **部署** | 库/自托管/云 | CLI/API/云 | 云为主（社区版停更）|
| **延迟** | ~1s（提取时）| 依赖 LLM 响应 | <200ms（检索时）|

---

## 四、各自的核心差异点

**mem0**：最轻量接入，一行代码加记忆。适合快速给现有 AI 应用加个性化层，不需要改 Agent 架构。

**Letta**：记忆是 Agent 的一部分，Agent 自己管理自己的记忆。适合构建"有自我意识、能学习进化"的 Agent，记忆和 Agent 耦合较深。

**Zep**：强调时间维度和关系网络，适合业务复杂度高、数据来源多元的场景（客服、医疗、CRM 等），走企业级路线。

---

## 五、和 onlonlonl 伴生工具的关系

这三个框架解决的都是「AI 记住用户」，但都是**对话记忆**——从非结构化对话里提取信息。

onlonlonl 的方案是另一个维度：**结构化场景记录**。用户主动输入（弹琴、记账、喝酒），数据有明确 schema，通过 SQL 精确查询，不需要语义理解。

两种路径不互斥——可以把 mem0/Letta/Zep 理解为「AI 的隐性记忆」（从对话中自动沉淀），onlonlonl 的工具是「AI 的显性档案」（用户主动建立）。

---

## 六、延伸阅读

- Graphiti（Zep 底层）：https://github.com/getzep/graphiti
- Letta 模型排行榜：https://leaderboard.letta.com/
- mem0 论文：arXiv:2504.19413
