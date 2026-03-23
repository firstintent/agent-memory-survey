# 全项目对比表

## 独立记忆框架

| 项目 | ⭐ Stars | License | 架构模式 | 记忆类型 | 存储后端 | 检索方式 | CLI集成 | LongMemEval | 部署 | 成熟度 |
|------|---------|---------|---------|---------|---------|---------|---------|-------------|------|--------|
| **Mem0** | ~50.2k | Apache 2.0 | 向量+图混合 | E,S | 向量DB+Neo4j/FalkorDB | 向量+图 | ⚠️理论可接 | 49% | 本地/SaaS | 生产 |
| **Letta** | ~21.6k | Apache 2.0 | 自编辑记忆 | E,S,W | 自有存储 | Agent工具 | ✅原生最强 | — | 本地/Docker | Beta |
| **Supermemory** | ~15.9k | 开源 | LLM-Native | E,S | LLM压缩 | LLM判断 | ⚠️有插件 | 99% | SaaS/本地 | Beta |
| **Weaviate** | ~15.8k | BSD-3 | 向量DB | S | 向量DB | 向量+BM25 | ❌基础设施 | — | 本地/云 | 生产 |
| **Memvid** | ~13.5k | MIT | 单文件 | E,S | .mv2文件 | BM25+向量 | ❌ | LoCoMo+35% | 本地 | Beta |
| **Cognee** | ~13k | MIT | ECL+知识图谱 | E,S | 图DB+向量 | 图+向量 | ❌ | — | 本地/Docker | Beta |
| **Qdrant** | ~29k | Apache 2.0 | 向量DB | S | 向量DB | HNSW | ❌基础设施 | — | 本地/云 | 生产 |
| **ChromaDB** | ~26.7k | Apache 2.0 | 向量DB | S | 向量DB | 向量 | ❌基础设施 | — | 本地/云 | 生产 |
| **MemOS** | ~7.4k | 开源 | 记忆OS | E,S,P | 内置 | 技能匹配 | ❌ | — | 本地 | Beta |
| **Hindsight** | ~5.6k | MIT | 时序图谱 | E,S,W | 混合 | 向量+BM25+图+时间 | ✅推荐 | 91.4% | 本地 | Beta |
| **Zep** | ~4.2k | 开源 | 时序知识图谱 | E,S | 图DB | 图+向量 | ⚠️理论可接 | 71% | 本地/云 | Beta |
| **SimpleMem** | ~2.8k | 开源 | 语义压缩 | E,S | 压缩存储 | 意图感知 | ❌ | LoCoMo+26.4% | 本地 | 研究 |
| **Memento** | ~2.2k | 开源 | 持续学习 | E,S | — | — | ❌ | — | — | 研究 |
| **claude-subconscious** | ~862 | MIT | Letta注入 | E,S | Letta后端 | Letta检索 | ✅Claude专用 | — | 本地 | Beta |
| **SuperLocalMemory** | — | 开源 | 信息几何 | E,S | 本地文件 | 向量+BM25 | ⚠️MCP | 74.8%(local) | 纯本地 | Beta |
| **A-MEM** | — | 开源 | Zettelkasten | E,S | ChromaDB | 向量+链接 | ❌ | — | 本地 | 研究 |
| **Google Always-On** | — | MIT | LLM-Native | E,S | SQLite | LLM判断 | ❌ | — | 本地 | 示例 |
| **RetainDB** | — | — | — | — | — | — | ❌ | 88%* | — | 新项目 |
| **memory-bridge** | — | — | 同步工具 | — | — | — | ✅双向同步 | — | 本地 | 新项目 |
| **MemMachine** | — | Apache 2.0 | 双存储 | E,S | 图+SQL | 图+SQL | ❌ | — | 本地/Docker | Beta |

> 记忆类型: E=Episodic, S=Semantic, P=Procedural, W=Working
> *RetainDB 88% 为 preference recall 指标

## Agent 自带记忆系统

| Agent | 存储 | 自动记忆 | 语义检索 | 用户控制 | 版本控制 | 持久范围 | 记忆模式 |
|-------|------|---------|---------|---------|---------|---------|---------|
| **Claude Code** | 文件 (CLAUDE.md) | ✅ | ❌ | ✅完全 | ✅ | 跨会话 | 文件持久化 |
| **Cursor** | 文件 (.cursor/rules/) | ❌ | ❌ | ✅完全 | ✅ | 跨会话 | 文件持久化 |
| **Codex CLI** | SQLite+文件 | ✅ | ❌ | ✅完全 | ✅ | 跨会话 | 结构化存储 |
| **Windsurf** | 文件+云端 | ✅ | ❌ | ✅完全 | ⚠️ | 跨会话 | 混合方式 |
| **Augment Code** | 应用记忆 | ✅ | ❌ | ✅Review | ❌ | 跨会话 | 混合方式 |
| **Devin** | 应用KB | ✅ | ❌ | ✅完全 | ❌ | 跨会话 | 知识库 |
| **Kiro** | 文件 (.amazonq/) | ✅ | ❌ | ✅完全 | ✅ | 跨会话 | 文件持久化 |
| **Cline** | 项目Markdown | ❌ | ❌ | ✅完全 | ✅ | 跨会话 | 文件持久化 |
| **Aider** | LanceDB | ✅ | ✅ | ⚠️部分 | ❌ | 跨会话 | 向量嵌入 |
| **GitHub Copilot** | 云端(repo) | ✅ | ❌ | ⚠️有限 | ❌ | 跨会话 | 云端记忆 |
| **ChatGPT** | 云端(user) | ✅ | ❌ | ✅完全 | ❌ | 跨会话 | 云端记忆 |
| **Gemini** | 云端(user) | ✅ | ❌ | ⚠️部分 | ❌ | 跨会话 | 云端记忆 |
| **Replit Agent** | 内存 | ✅ | ❌ | ⚠️有限 | ❌ | 仅会话 | 轨迹压缩 |

## 按热度排序 (2026.03 X)

| 排名 | 项目 | X热度指标 | 关键词 |
|------|------|----------|--------|
| 1 | Hindsight | 433赞/29k浏览帖 | "杀了RAG", SOTA |
| 2 | Supermemory | 99%重磅帖 | "向量DB已死" |
| 3 | Letta | 多条实战帖 | "日常驱动", CLI标配 |
| 4 | RetainDB | 88%新帖 | 3月黑马 |
| 5 | Memento | 高赞帖 | 持续学习 |
| 6 | Mem0 | 对比讨论 | "老将" |
| 7 | Zep | 被当参照 | "旧时代" |
