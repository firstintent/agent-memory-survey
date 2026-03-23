# Agent Memory Systems Survey (2026.03)

AI Agent 记忆系统全景调研 — 独立记忆框架 + Agent 自带记忆，覆盖 30+ 个项目 × 15 个维度。

> **核心趋势**: 2026 年 3 月，向量记忆被 agentic/观测记忆全面碾压，进入"非向量时代"。

## 🔥 2026.03 热度排行

| 排名 | 项目 | 亮点 | LongMemEval | Stars |
|------|------|------|-------------|-------|
| 🥇 | [Hindsight](projects/hindsight.md) | TEMPR架构, "杀了RAG" | 91.4% | ~5.6k |
| 🥈 | [Supermemory](projects/supermemory.md) | LLM压缩, "向量DB已死" | 99% | ~15.9k |
| 🥉 | [Letta](projects/letta.md) | CLI实用王, /remember | — | ~21.6k |
| 4 | [Mem0](projects/mem0.md) | 老将, 图+向量 | 49% | ~50.2k |
| 5 | [Zep](projects/zep.md) | 时序图谱, "旧时代"参照 | 71% | ~4.2k |
| 🔌 | [Claude-Mem](projects/claude-mem.md) | Claude Code插件, AI压缩+混合检索 | — | ~39.7k |
| 🆕 | [RetainDB](projects/retaindb.md) | 88% preference recall | 88%* | — |
| 🆕 | [Memento](projects/memento.md) | 无微调持续学习 | — | ~2.2k |

## 📖 目录

### 核心文档

| 文档 | 说明 |
|------|------|
| [dimensions.md](dimensions.md) | 15 个调研维度详解 |
| [comparison-table.md](comparison-table.md) | 全项目 × 全维度对比表 |
| [architecture-patterns.md](architecture-patterns.md) | 6 大架构模式 (向量→agentic 演进) |
| [benchmarks.md](benchmarks.md) | LongMemEval/LoCoMo/MemoryAgentBench |
| [cli-integration.md](cli-integration.md) | Claude Code & Codex CLI 集成实战 |
| [selection-guide.md](selection-guide.md) | 按场景选型建议 |
| [self-build-guide.md](self-build-guide.md) | 🔥 自研记忆系统完整建议 |
| [builtin-comparison.md](builtin-comparison.md) | Agent 自带记忆横向对比 |
| [references.md](references.md) | 论文/博客/X帖/awesome-list |

### 独立记忆框架 (projects/)

| Tier | 项目 | 架构 | Stars |
|------|------|------|-------|
| **T1 爆款** | [Hindsight](projects/hindsight.md) | 时序图谱 TEMPR+CARA | ~5.6k |
| **T1 爆款** | [Supermemory](projects/supermemory.md) | LLM-Native 压缩 | ~15.9k |
| **T1 爆款** | [Letta](projects/letta.md) | 自编辑记忆 | ~21.6k |
| T2 | [Mem0](projects/mem0.md) | 向量+图混合 | ~50.2k |
| T2 | [Zep](projects/zep.md) | 时序知识图谱 | ~4.2k |
| T2 | [Cognee](projects/cognee.md) | ECL+知识图谱 | ~13k |
| T2 | [SimpleMem](projects/simplemem.md) | 语义无损压缩 | ~2.8k |
| T2 | [Memvid](projects/memvid.md) | 单文件 .mv2 | ~13.5k |
| T2 | [SuperLocalMemory](projects/superlocalmemory.md) | 信息几何+纯本地 | — |
| T2 | [A-MEM](projects/a-mem.md) | Zettelkasten | — |
| T2 | [MemMachine](projects/memmachine.md) | 多LLM双存储 | — |
| T2 | [MemOS](projects/memos.md) | AI记忆OS | ~7.4k |
| T2 | [Google Always-On](projects/google-always-on.md) | LLM-Native+SQLite | — |
| T3 新星 | [RetainDB](projects/retaindb.md) | 88% SOTA | — |
| T3 新星 | [Memento](projects/memento.md) | 持续学习 | ~2.2k |
| CC插件 | [Claude-Mem](projects/claude-mem.md) | AI压缩+混合检索 | ~39.7k |
| CLI工具 | [memory-bridge](projects/memory-bridge.md) | CLI记忆同步 | — |
| CLI工具 | [claude-subconscious](projects/claude-subconscious.md) | Letta→Claude注入 | ~862 |

### Agent 自带记忆 (builtin/)

| Agent | 记忆模式 | 详情 |
|-------|---------|------|
| [Claude Code](builtin/claude-code.md) | 文件持久化 (CLAUDE.md + auto-memory) | ✅ |
| [Cursor](builtin/cursor.md) | 文件持久化 (.cursor/rules/) | ✅ |
| [GitHub Copilot](builtin/github-copilot.md) | 云端 (repo-scoped) | ✅ |
| [Windsurf](builtin/windsurf.md) | 混合 (rules + cascade) | ✅ |
| [Aider](builtin/aider.md) | 向量嵌入 (LanceDB) | ✅ |
| [Devin](builtin/devin.md) | 知识库 + playbooks | ✅ |
| [Codex CLI](builtin/codex-cli.md) | 结构化 (SQLite + MEMORY.md) | ✅ |
| [ChatGPT](builtin/chatgpt.md) | 云端 saved memories | ✅ |
| [Gemini](builtin/gemini.md) | Personal Intelligence | ✅ |
| [Kiro](builtin/kiro.md) | 文件持久化 (原 Amazon Q) | ✅ |
| [Augment Code](builtin/augment-code.md) | Memory Review | ✅ |
| [Cline](builtin/cline.md) | Memory Bank 方法论 | ✅ |
| [Replit Agent](builtin/replit-agent.md) | 轨迹压缩 | ✅ |

## 🏗 自研参考架构

基于 2026.03 趋势的推荐架构:

```
┌────────────────────────────┐
│     MCP Server 接口层       │  ← Claude Code/Codex CLI 原生
├────────────────────────────┤
│   自编辑工具层 (Letta式)    │  ← Agent 主动管理
├────────────────────────────┤
│  LLM-Native (Supermemory式) │  ← 99% 准确率, 无向量
├────────────────────────────┤
│  时间推理 (Hindsight TEMPR) │  ← 冲突消解, 遗忘
├────────────────────────────┤
│   SQLite + 文件混合存储     │  ← 零依赖, 可git
└────────────────────────────┘
```

## 数据来源

- GitHub 仓库和文档
- 学术论文 (arXiv, NeurIPS, ACM)
- 2026 年 3 月 X/Twitter 实时讨论
- 技术博客和对比文章

详见 [references.md](references.md)。

---

*调研时间: 2026 年 3 月 23 日*
