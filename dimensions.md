# 调研维度详解

本文档定义了用于评估和对比 Agent 记忆系统的 15 个维度。

## 1. 记忆类型

AI Agent 记忆对应人类认知科学的四种记忆类型：

| 类型 | 说明 | 示例 |
|------|------|------|
| **Episodic (情景记忆)** | 存储具体事件、交互和经历，带时间戳 | "用户在 3/15 修复了登录 bug" |
| **Semantic (语义记忆)** | 组织化的知识库：事实、概念、关系 | "项目使用 PostgreSQL + Redis" |
| **Procedural (程序性记忆)** | 工作流、技能、决策逻辑 | "部署流程：test → build → deploy" |
| **Working (工作记忆)** | 当前活跃上下文、对话状态 | 当前对话中的变量和目标 |

**自研关键问题**：是否需要全部四种？大多数系统只实现 Semantic + Episodic。

## 2. 存储后端

| 后端 | 优势 | 劣势 | 代表系统 |
|------|------|------|---------|
| **向量数据库** | 语义相似搜索 | 无关系建模 | Mem0, ChromaDB, Qdrant |
| **图数据库** | 关系/时间推理 | 复杂度高 | Zep, Cognee (Neo4j) |
| **关系型数据库** | 成熟、结构化 | 语义搜索弱 | Google Always-On (SQLite) |
| **文件系统** | 零依赖、可版本控制 | 检索能力有限 | Claude Code, Cursor, Cline |
| **单文件** | 极致便携 | 容量限制 | Memvid (.mv2) |
| **云端** | 跨设备、团队共享 | 隐私风险 | ChatGPT, GitHub Copilot |
| **LLM 压缩** | 无向量开销 | 依赖 LLM 质量 | Supermemory |
| **混合** | 各取所长 | 架构复杂 | Hindsight, Mem0 |

**2026.03 趋势**：纯向量方案被 LLM-native 和混合方案超越。

## 3. 检索方式

| 方式 | 原理 | 适用场景 |
|------|------|---------|
| **向量相似度** | embedding cosine similarity | 语义模糊匹配 |
| **BM25** | 词频统计 | 精确关键词匹配 |
| **图遍历** | 关系路径搜索 | 多跳推理 |
| **时间加权** | 结合 recency + importance | 时序敏感场景 |
| **混合检索** | 多种方式加权融合 | 通用场景 |
| **LLM-native** | LLM 直接判断相关性 | 无向量基础设施时 |

**SOTA 趋势**：Hindsight 的混合检索 (向量 + BM25 + 图 + 时间) 达到 91.4%。

## 4. 适配 Agent / CLI 集成

评估维度：
- **框架集成**：LangChain, CrewAI, LlamaIndex, Anthropic SDK
- **CLI 集成**：Claude Code, Codex CLI, Cursor, Aider
- **协议支持**：MCP (Model Context Protocol), REST API, Python SDK
- **集成深度**：原生支持 vs SDK 调用 vs 手动配置

**2026.03 实况**：仅 Letta 和 Hindsight 有 Claude Code/Codex CLI 实战用户。

## 5. GitHub Stars

社区热度指标，反映关注度但不等于质量。需结合以下看：
- Star 增长趋势（是否在加速）
- Issue/PR 活跃度
- Contributor 数量
- 最近 commit 时间

## 6. License

| 类型 | 含义 | 注意事项 |
|------|------|---------|
| MIT | 最宽松 | 可商用，可闭源 |
| Apache 2.0 | 宽松 + 专利保护 | 可商用 |
| AGPL | 强 copyleft | 网络服务也需开源 |
| BSL / SSPL | 源码可用但限制商用 | 不算真正开源 |
| 闭源 | 仅 SaaS | 无法自部署 |

## 7. 部署方式

- **本地运行**：pip install / npm install，零外部依赖
- **Docker**：容器化部署，适合团队
- **SaaS**：托管服务，开箱即用
- **嵌入式**：作为库集成到代码中

## 8. 隐私合规

- **纯本地**：数据不离开机器 (SuperLocalMemory, Memvid)
- **自托管**：数据在自己服务器 (Mem0 OSS, Letta)
- **云端**：数据在第三方 (Mem0 Cloud, ChatGPT)
- **合规认证**：GDPR, HIPAA, EU AI Act

## 9. 多 Agent 支持

- **共享记忆**：多个 Agent 访问同一知识库
- **记忆隔离**：Agent 间记忆隔离，防止信息泄漏
- **协作学习**：Agent 互相丰富集体知识
- **同步机制**：分布式记忆的一致性保证

## 10. 时间推理

- **时序知识**：知道事实发生的时间和顺序
- **冲突消解**：新信息覆盖旧信息 ("搬到上海" 替代 "住在北京")
- **选择性遗忘**：自动淘汰过时/不相关信息
- **因果推理**：理解事件的因果关系

## 11. 性能基准

| 基准 | 测试内容 | 领先者 |
|------|---------|--------|
| **LongMemEval** | 长期记忆评估 | Supermemory 99%, Hindsight 91.4% |
| **LoCoMo** | 长对话记忆 | SimpleMem F1 +26.4%, Memvid +35% |
| **MemoryAgentBench** | 4 项核心能力 | (多系统) |
| **AMA-Bench** | 长周期 Agent 记忆 | (新基准) |

## 12. Token 效率

衡量记忆系统对 LLM API 调用量的影响：
- **Token 节省倍数**：SimpleMem 30x, SuperLocalMemory 60% zero-LLM
- **上下文注入开销**：注入记忆占用多少 context window
- **API 调用频率**：检索/存储操作是否需要额外 LLM 调用

## 13. 多模态

支持的输入类型：
- 文本 (所有系统)
- 图片 (Google Always-On, MemOS)
- 音频 (Google Always-On)
- 视频 (Google Always-On, Memvid)
- PDF (Supermemory, Google Always-On)
- 代码 (大部分系统)

## 14. 成熟度

| 级别 | 定义 | 示例 |
|------|------|------|
| **生产就绪** | 有企业用户、SLA、文档完善 | Mem0, ChromaDB |
| **Beta** | 功能完整但仍在迭代 | Letta, Hindsight |
| **研究原型** | 论文配套代码，非生产级 | A-MEM, SimpleMem |
| **新项目** | 刚发布，待验证 | RetainDB, Memento |

## 15. API / SDK

- **REST API**：HTTP 调用，语言无关
- **Python SDK**：pip install，最常见
- **Node.js SDK**：npm install
- **MCP Server**：Model Context Protocol，Claude Code 原生支持
- **CLI**：命令行工具
- **SDK 质量**：文档、类型注解、错误处理
