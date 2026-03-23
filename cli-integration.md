# Claude Code & Codex CLI 集成实战

> 基于 2026 年 3 月 X 实际用户帖子，非官网宣传。

## 集成现状总览

| 记忆系统 | Claude Code | Codex CLI | 集成方式 | X 实战帖 |
|---------|------------|-----------|---------|---------|
| **Letta** | ✅ 原生最强 | ✅ 原生最强 | CLI + MCP + subagent | 大量 |
| **Hindsight** | ✅ 推荐 | ⚠️ 理论可接 | Claude Agent SDK | 多条 |
| **memory-bridge** | ✅ 专用 | ✅ 专用 | 双向同步工具 | 有 |
| **claude-subconscious** | ✅ 专用 | ❌ | Letta agent 注入 | 有 |
| Mem0 | ⚠️ SDK 可接 | ⚠️ SDK 可接 | Python SDK / MCP | 几乎没有 |
| Supermemory | ⚠️ 有插件 | ⚠️ SDK 可接 | 内置插件 | 几乎没有 |
| 其他 | ⚠️ 理论可接 | ⚠️ 理论可接 | MCP / SDK | 无 |

## Letta (Letta Code) — CLI 集成之王

### 为什么是最强？

X 用户真实反馈：
- "我日常就是 Letta Code + Codex 5.3"
- "Codex CLI harness 直接跑 Letta"
- "Claude Code 里用 Letta 的 /remember 和 skill-creator"

### 集成方式

**1. Letta Code CLI (独立使用)**

```bash
# 安装 Letta
pip install letta

# 启动 Letta Code
letta code

# 在 Letta Code 中使用记忆
/remember "这个项目使用 FastAPI + PostgreSQL"
/skills list
```

**2. 与 Claude Code 配合**

通过 `claude-subconscious` 项目：

```bash
# 安装 claude-subconscious (Letta agent)
# 自动注入记忆到 Claude Code
pip install claude-subconscious

# 配置
claude-subconscious init --claude-code

# 运行：Letta agent 在后台自动管理 Claude Code 的记忆
claude-subconscious run
```

**3. 通过 MCP Server**

```json
// Claude Code settings.json
{
  "mcpServers": {
    "letta": {
      "command": "letta",
      "args": ["mcp-server"]
    }
  }
}
```

### 核心能力

- `/remember` — 明确记住某条信息
- `skill-creator` — 从经验中学习可复用技能
- `subagents` — 子代理共享记忆
- 记忆桥接 — 在不同 CLI 间同步记忆

---

## Hindsight — Claude Code 用户最推荐

### X 用户推荐

- "用 Claude Code 建 agent？赶紧上 Hindsight 提升记忆"
- "直接接 Claude Agent SDK"

### 集成方式

**通过 Claude Agent SDK**

```python
from hindsight import HindsightMemory
from anthropic import Anthropic

# 初始化 Hindsight
memory = HindsightMemory(
    storage_backend="sqlite",  # 或 postgres
)

# 在 Claude Agent 中使用
client = Anthropic()

# 保留记忆
memory.retain("用户偏好使用 TypeScript", context="code_review")

# 召回记忆
relevant = memory.recall("用户的编程语言偏好")

# 反思 (后台整合)
memory.reflect()
```

**通过 MCP Server**

```json
{
  "mcpServers": {
    "hindsight": {
      "command": "hindsight-mcp",
      "args": ["--storage", "sqlite"]
    }
  }
}
```

---

## memory-bridge — 双向同步

专门解决 Codex CLI 和 Claude Code 之间的记忆同步问题。

```bash
# 安装
pip install memory-bridge

# 配置双向同步
memory-bridge config \
  --codex-memory ~/.codex/MEMORY.md \
  --claude-memory ~/.claude/projects/*/memory/

# 运行同步
memory-bridge sync

# 可搭配任意记忆系统
memory-bridge sync --backend hindsight
memory-bridge sync --backend letta
```

---

## 通用集成模式

对于没有原生 CLI 集成的记忆系统，可以通过以下方式接入：

### 方式 1: MCP Server

大多数记忆系统都可以包装为 MCP Server，Claude Code 原生支持 MCP。

```python
# 将任意记忆系统包装为 MCP Server
from mcp import Server

server = Server("memory-system")

@server.tool("remember")
def remember(content: str) -> str:
    memory_system.store(content)
    return "Stored"

@server.tool("recall")
def recall(query: str) -> str:
    results = memory_system.search(query)
    return format_results(results)
```

### 方式 2: Claude Code Hooks

利用 Claude Code 的 hooks 系统，在特定事件时自动存取记忆。

```json
// settings.json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "command": "python -c 'from my_memory import save; save()'"
    }]
  }
}
```

### 方式 3: CLAUDE.md 自动注入

将记忆系统的输出写入 CLAUDE.md 或 memory/ 目录。

```bash
# 定期将记忆系统的内容同步到 CLAUDE.md
my-memory-system export --format claude-md > .claude/CLAUDE.md
```

---

## 选型建议 (CLI 用户)

| 场景 | 推荐 | 原因 |
|------|------|------|
| **日常编码** | Letta | 最丝滑的 CLI 集成, /remember + 技能学习 |
| **追求准确率** | Hindsight | 91.4% SOTA, Claude Agent SDK 原生 |
| **同时用 Codex + Claude** | memory-bridge + Letta/Hindsight | 双向同步 |
| **零依赖** | Claude Code 原生 | CLAUDE.md + auto-memory 已够用 |
| **团队共享记忆** | Mem0 / Hindsight + MCP | 需要服务端 |
