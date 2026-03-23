# 架构模式：从向量到 Agentic 的演进

## 演进时间线

```
2023          2024          2025          2026.03
  │             │             │             │
  ▼             ▼             ▼             ▼
向量RAG时代    图+向量混合    LLM-native    "非向量时代"
ChromaDB      Mem0 Graph    Supermemory   Hindsight SOTA
Pinecone      Zep temporal  Google SQLite  RetainDB
              Cognee ECL    SimpleMem      观测记忆
```

## 六大架构模式

### 模式 1: 纯向量 (Vector-Only)

```
用户输入 → Embedding → 向量DB → Top-K 相似 → 注入 Context
```

**代表**: ChromaDB, Qdrant, 早期 Mem0

**原理**: 将文本转为向量嵌入，通过余弦相似度检索最相关的记忆片段。

**优势**: 简单、成熟、语义理解好
**劣势**: 无关系建模、无时间推理、容易检索到"相似但不相关"的内容

**2026 现状**: 被视为"旧时代"方案，LongMemEval 上表现最差 (Mem0 49%)。

---

### 模式 2: 图 + 向量混合 (Graph-Augmented)

```
用户输入 → 实体抽取 → 知识图谱 (Neo4j/FalkorDB)
          → Embedding → 向量DB
          → 混合检索 (图遍历 + 向量相似)
```

**代表**: Mem0 Graph, Zep, Cognee

**原理**: 在向量检索基础上，构建实体-关系知识图谱，支持多跳推理和关系查询。

**Cognee ECL 流水线**:
1. **Extract**: 从文档中提取实体和关系
2. **Cognify**: 构建三元组 (主体-关系-客体)
3. **Load**: 存入图数据库 + 向量存储

**优势**: 支持关系推理、实体追踪、多跳查询
**劣势**: 图构建开销大、图谱维护复杂、仍依赖向量

---

### 模式 3: 时序知识图谱 (Temporal Knowledge Graph)

```
用户输入 → 实体 + 时间戳 → 时序图谱
          → 时间加权检索
          → 冲突消解 (新事实覆盖旧事实)
```

**代表**: Zep, Hindsight

**原理**: 在知识图谱上增加时间维度，追踪实体状态随时间的变化。

**Hindsight TEMPR 架构**:
- **T**emporal **E**ntity **M**emory **P**riming **R**etrieval
- 4 个记忆网络: 世界事实、Agent 经验、实体摘要、演化信念
- 3 个操作: retain (保留)、recall (召回)、reflect (反思)
- 混合检索: 向量 + BM25 + 图遍历 + 时间过滤

**优势**: 时间推理、冲突消解、选择性遗忘
**劣势**: 架构复杂、部署门槛高

---

### 模式 4: LLM-Native (无向量)

```
用户输入 → LLM 提取关键信息 → 结构化存储 (SQLite/文件)
新查询 → LLM 判断相关性 → 直接返回
```

**代表**: Supermemory, Google Always-On, Codex CLI

**原理**: 完全依赖 LLM 理解和压缩信息，不需要 embedding 模型和向量数据库。

**Supermemory 方法**:
- LLM 压缩取代向量嵌入
- 自动事实抽取和去重
- 处理信息更新 ("搬到上海" 替代 "住在北京")
- 99% LongMemEval (远超向量方案)

**Google Always-On 方法**:
- IngestAgent: 多模态内容提取
- ConsolidationAgent: 后台记忆整合
- QueryAgent: 从记忆合成答案
- 存储: 纯 SQLite，零向量基础设施

**优势**: 零向量基础设施、成本低、理解力强
**劣势**: 依赖 LLM 质量、每次检索需 LLM 调用

---

### 模式 5: 自编辑记忆 (Self-Editing Memory)

```
Agent 运行 → 使用 memory_write() 工具主动写入记忆
          → 使用 memory_read() 工具主动检索
          → 自主决定保留/更新/删除
```

**代表**: Letta (MemGPT), A-MEM

**原理**: 给 Agent 提供记忆管理工具，让 Agent 自主决定什么值得记住。

**Letta 架构**:
- Agent 拥有 `core_memory_append`, `core_memory_replace`, `archival_memory_search` 等工具
- 区分 in-context memory (核心记忆块) vs archival memory (长期存储)
- Agent 像人一样主动"记笔记"和"翻笔记"

**A-MEM Zettelkasten 方法**:
- 借鉴卢曼卡片盒方法
- 每条记忆是一张"卡片"，带索引和链接
- 新记忆触发对相关旧记忆的更新
- 记忆网络自动演化

**优势**: Agent 有自主权、灵活度高、适合长期运行的 Agent
**劣势**: 依赖 Agent 的判断力、可能遗漏重要信息

---

### 模式 6: 文件持久化 (File-Based Persistence)

```
规则/记忆 → Markdown/YAML 文件 → 加载到 Context Window
```

**代表**: Claude Code (CLAUDE.md), Cursor (.cursor/rules), Cline, Kiro

**原理**: 将记忆存储为项目中的配置文件，启动时加载到上下文。

**Claude Code 实现**:
- `CLAUDE.md`: 手动编写的项目规则和约定
- `~/.claude/projects/memory/`: 自动记忆 (用户偏好、反馈、项目上下文)
- `MEMORY.md`: 记忆索引文件
- 每条记忆带 frontmatter (name, type, description)

**优势**: 零依赖、可版本控制、用户完全可控、透明
**劣势**: 不支持语义检索、容量受限于 context window、手动维护成本

---

## 架构模式对比

| 模式 | 检索质量 | 复杂度 | 部署成本 | Token 效率 | 代表基准 |
|------|---------|--------|---------|-----------|---------|
| 纯向量 | ★★☆ | ★☆☆ | ★★☆ | ★★★ | Mem0 49% |
| 图+向量 | ★★★ | ★★★ | ★★★ | ★★☆ | Zep 71% |
| 时序图谱 | ★★★★ | ★★★★ | ★★★ | ★★☆ | Hindsight 91.4% |
| LLM-Native | ★★★★★ | ★★☆ | ★☆☆ | ★☆☆ | Supermemory 99% |
| 自编辑 | ★★★ | ★★☆ | ★★☆ | ★★☆ | Letta (实用) |
| 文件持久化 | ★☆☆ | ★☆☆ | ☆☆☆ | ★★★★ | Claude Code |

## 自研建议

根据 2026.03 趋势，推荐的架构组合：

1. **基础层**: LLM-Native 压缩 (参考 Supermemory)，避免向量基础设施
2. **增强层**: 时间推理 (参考 Hindsight TEMPR)，支持冲突消解
3. **接口层**: 自编辑工具 (参考 Letta)，让 Agent 主动管理记忆
4. **持久层**: 文件 + SQLite 混合 (参考 Codex CLI)，可版本控制
5. **协议层**: MCP Server，原生支持 Claude Code/Codex CLI
