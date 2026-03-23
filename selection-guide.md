# 按场景选型建议

## 快速选型矩阵

| 你的场景 | 首选 | 次选 | 避免 |
|---------|------|------|------|
| **Claude Code/Codex CLI 日常** | Letta | Hindsight | 纯向量方案 |
| **追求 SOTA 准确率** | Supermemory | Hindsight | Mem0, Zep |
| **纯本地/隐私敏感** | SuperLocalMemory | Memvid | 任何 SaaS |
| **多 Agent 协作** | Hindsight | Mem0 | 文件方案 |
| **零基础设施** | Memvid | Google Always-On | 需要图DB的方案 |
| **企业/合规** | SuperLocalMemory | Mem0 自托管 | ChatGPT 记忆 |
| **研究/论文** | A-MEM | SimpleMem | 闭源方案 |
| **自研参考** | Hindsight + Supermemory | Letta | — |

## 按角色推荐

### 独立开发者 / Solopreneur

**推荐**: Letta + Claude Code 原生记忆

理由:
- Letta 的 /remember + skill-creator 覆盖日常需求
- Claude Code 的 CLAUDE.md + auto-memory 零成本
- 不需要部署数据库或服务

### 小团队 (2-10 人)

**推荐**: Hindsight + MCP

理由:
- 91.4% 准确率，值得维护成本
- MCP 协议让团队成员的 Claude Code 共享记忆
- 支持多 Agent 共享

### 企业 / 合规场景

**推荐**: SuperLocalMemory 或 Mem0 自托管

理由:
- SuperLocalMemory: GDPR/HIPAA/EU AI Act, 数据不离开机器
- Mem0 自托管: 成熟的企业特性, 图+向量

### 自研记忆系统

**推荐架构** (2026.03 最佳实践):

```
┌─────────────────────────────────────┐
│          MCP Server 接口层           │  ← Claude Code/Codex CLI 原生接入
├─────────────────────────────────────┤
│      自编辑工具层 (参考 Letta)       │  ← Agent 主动管理记忆
├─────────────────────────────────────┤
│   LLM-Native 压缩 (参考 Supermemory)│  ← 不依赖向量, 99% 准确率
├─────────────────────────────────────┤
│   时间推理层 (参考 Hindsight TEMPR)  │  ← 冲突消解, 选择性遗忘
├─────────────────────────────────────┤
│     SQLite + 文件混合存储            │  ← 零外部依赖, 可版本控制
└─────────────────────────────────────┘
```

关键设计决策:
1. **不用向量 DB** — LLM-native 方案已证明优于向量 (99% vs 49%)
2. **MCP 优先** — 原生接入 Claude Code/Codex CLI 生态
3. **文件 + SQLite** — 零依赖, 可 git 管理
4. **TEMPR 模式** — retain/recall/reflect 三操作覆盖记忆生命周期
5. **自编辑接口** — 让 Agent 像人一样主动"记笔记"

## 迁移路径

### 从 Mem0 迁移

```
Mem0 (向量+图) → Hindsight (时序图谱) → 自研 (LLM-native)
```

1. 导出 Mem0 的记忆为 JSON
2. Hindsight retain() 批量导入
3. 逐步替换为自研系统

### 从 Claude Code 原生记忆扩展

```
CLAUDE.md → + Letta (自编辑) → + Hindsight (SOTA 检索)
```

1. 保留 CLAUDE.md 作为项目规则
2. 加 Letta 做动态记忆管理
3. 需要更高准确率时加 Hindsight

## 不推荐的选择

| 场景 | 不推荐 | 原因 |
|------|--------|------|
| 新项目 | Mem0 | 49% 基准, 被新方案超越 |
| CLI 集成 | Zep | 无 CLI 实战, 71% 偏低 |
| 隐私敏感 | ChatGPT/Gemini | 数据在第三方云端 |
| 生产环境 | A-MEM/SimpleMem | 研究原型, 非生产级 |
