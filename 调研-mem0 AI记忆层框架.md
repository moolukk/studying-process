# 调研：mem0 — AI Agent 的通用记忆层

> 调研时间：2026-04-27
> 来源：https://github.com/mem0ai/mem0
> 背景：YC S24，2023年6月开源，目前最活跃的 AI 记忆基础设施项目之一

---

## 一、是什么

mem0（读作 "mem-zero"）是一个给 AI Agent 和 AI 助手加装「长期记忆」的基础设施层。

核心问题：LLM 本身没有跨会话记忆，每次对话从零开始。mem0 的作用是在对话之外维护一个结构化的记忆库，让 AI 在下次对话时能「想起」用户的偏好、历史、上下文。

**定位：** 不是面向用户的产品，而是面向开发者的记忆中间件，嵌入到 AI 应用里使用。

---

## 二、核心架构

```
对话输入
   ↓
[LLM 提取] ← 从对话中抽取事实（单次调用，只 ADD 不 UPDATE）
   ↓
[实体链接] ← 抽取实体，跨记忆建立关联
   ↓
[向量存储] ← 记忆条目向量化存入
   ↓
[多信号检索] ← 语义向量 + BM25 关键词 + 实体匹配，并行打分融合
   ↓
检索结果注入下次对话的 system prompt
```

**模块拆解（代码目录）：**

| 目录 | 职责 |
|------|------|
| `mem0/memory/` | 核心记忆管理逻辑 |
| `mem0/llms/` | LLM 适配层（OpenAI、Anthropic 等）|
| `mem0/embeddings/` | 向量模型适配 |
| `mem0/vector_stores/` | 向量库适配（Qdrant、Pinecone 等）|
| `mem0/reranker/` | 检索后重排序 |
| `mem0/configs/` | 配置管理 |
| `mem0/client/` | 云服务 SDK |
| `openmemory/` | 自托管版本（含 Dashboard UI）|

---

## 三、2026 年 4 月的新算法（重大更新）

旧算法会对记忆做 UPDATE/DELETE，新算法改为**只 ADD，永不覆盖**：

| 变化点 | 旧算法 | 新算法 |
|------|------|------|
| 写入策略 | ADD + UPDATE + DELETE | 只 ADD，记忆累积 |
| LLM 调用次数 | 多次 | 单次（single-pass）|
| 实体处理 | 无 | 实体提取 + 跨记忆链接 |
| 检索方式 | 纯语义向量 | 语义 + BM25 + 实体，并行融合 |

**Benchmark 结果（同等模型栈）：**

| 基准 | 旧 | 新 |
|------|------|------|
| LoCoMo | 71.4 | **91.6** (+20) |
| LongMemEval | 67.8 | **93.4** (+26) |
| BEAM (1M token) | — | 64.1 |
| BEAM (10M token) | — | 48.6 |

平均延迟约 1 秒，token 消耗约 7K。已发 arXiv 论文（arXiv:2504.19413）。

---

## 四、部署方式

提供三种接入路径：

**1. Python/JS 库（最轻量）**
```bash
pip install mem0ai
npm install mem0ai
```
适合原型和测试，直连向量库，无 dashboard。

**2. 自托管服务器**
```bash
cd server && make bootstrap  # 启动 Qdrant + FastAPI + Next.js Dashboard
```
带权限管理，适合团队内部部署。

**3. 云服务（app.mem0.ai）**
零运维，API key 接入，适合生产。

---

## 五、记忆的三个层级

mem0 定义了三种记忆粒度，可分别存取：

- **User Memory** — 跨所有会话持久存在的用户偏好、背景信息
- **Session Memory** — 单次会话内的上下文
- **Agent Memory** — Agent 自身的状态和知识积累

---

## 六、生态集成

- **IDE 插件**：`.claude-plugin`、`.cursor-plugin` 目录（直接给 Claude Code / Cursor 加记忆）
- **框架**：LangGraph、CrewAI
- **前端**：Vercel AI SDK 集成
- **浏览器扩展**：跨 ChatGPT、Perplexity、Claude 页面注入记忆
- **CLI**：`mem0 add/search` 命令行管理记忆

---

## 七、和 onlonlonl 伴生工具的对比

二者都在解决「AI 记住用户」的问题，但路径完全不同：

| 维度 | mem0 | onlonlonl 系列 |
|------|------|------|
| 记忆形式 | 非结构化自然语言事实 | 结构化 SQL 数据（有明确表结构）|
| 存储后端 | 向量数据库（Qdrant 等）| Supabase Postgres |
| 检索方式 | 语义相似度 + 关键词 + 实体 | SQL 查询，精确 |
| 写入触发 | LLM 自动从对话中提取 | 用户主动操作 UI |
| 适用场景 | 通用对话记忆 | 特定生活场景的结构化数据 |
| 定位 | 基础设施中间件 | 终端用户工具 |
| 有无界面 | 有 dashboard，主要给开发者 | 有精心设计的专属 UI |

**关键差异**：mem0 记的是「Alice 喜欢深色模式」这种碎片化偏好；onlonlonl 记的是「这杯酒叫什么、几分、什么口感」这种完整结构化记录。前者靠相似度检索，后者靠 SQL 精确查询。两套方案适合不同需求，不互斥。

---

## 八、值得关注的点

- **「只 ADD 不 UPDATE」**是个有意思的设计选择，避免了记忆被错误覆盖，但会随时间膨胀，长期成本待观察
- **实体链接**让不同记忆条目之间建立关系图，接近知识图谱方向，比单纯的向量检索更有语义
- **Claude Plugin 目录**说明 mem0 正在积极进入 Claude 生态，未来可能和 MCP 有交集
- GitHub Star 量级很高，活跃度高，是这个赛道目前的标杆项目

---

## 九、项目信息

- **开源协议**：Apache 2.0
- **语言**：Python 主，TypeScript 次
- **论文**：arXiv:2504.19413
- **官网**：https://mem0.ai
- **GitHub**：https://github.com/mem0ai/mem0
