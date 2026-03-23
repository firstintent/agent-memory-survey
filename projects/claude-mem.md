# Claude-Mem

## 概要

| 项目 | Claude-Mem |
|------|-----------|
| GitHub | [thedotmack/claude-mem](https://github.com/thedotmack/claude-mem) |
| Stars | ~39.7k |
| License | AGPL-3.0 |
| 成熟度 | 生产就绪 |
| 类型 | Claude Code 插件 |

Claude-Mem 是 Claude Code 的记忆插件，通过自动捕获和压缩上下文，实现跨会话的记忆持久化。是 Claude Code 生态中 Stars 最高的记忆项目之一。

## 架构

```
Claude Code Session
    ├── Hook: session_start → 注入历史上下文
    ├── Hook: prompt_submit → 捕获用户意图
    ├── Hook: tool_use_complete → 捕获工具观测
    ├── Hook: session_end → 压缩总结
    └── Hook: stop → 保存状态

Claude-Mem 后台服务 (Bun, port 37777)
    ├── SQLite (持久化存储)
    ├── Chroma 向量数据库 (语义/混合检索)
    ├── AI 压缩引擎 (Claude agent-sdk)
    └── HTTP API (10 个搜索端点)
```

### 核心流程

1. **自动捕获**: 6 个生命周期 Hook 在关键时刻捕获工具观测和会话活动
2. **AI 压缩**: 使用 Claude agent-sdk 生成语义摘要，减少 ~10x token 开销
3. **语义检索**: 三层检索 — 索引搜索 → 时间线上下文 → 详细检索
4. **自动注入**: 新会话自动注入相关历史上下文

## 记忆类型

- ✅ Episodic (情景): 工具观测、会话记录
- ✅ Semantic (语义): AI 压缩的语义摘要
- ❌ Procedural (程序性): 不支持
- ✅ Working (工作): 自动注入的上下文

## 存储后端

- **SQLite**: 观测、会话、摘要的持久化存储
- **Chroma**: 向量数据库，支持语义搜索
- **配置**: `~/.claude-mem/settings.json`
- **全本地**: 数据不离开机器

## 检索方式

- **混合检索**: 关键词 + 语义嵌入
- **自然语言查询**: 通过 MCP 搜索
- **三层工作流**: 索引 → 时间线 → 详细
- **约 10x token 节省**

## Agent 集成

- **Claude Code 原生插件**: `/plugin install claude-mem`
- **Claude Desktop Skill**: 集成为桌面技能
- **MCP 协议**: 通过 MCP 提供搜索工具
- **OpenClaw 网关**: 支持云部署

### 安装

```bash
# Claude Code 插件安装
/plugin install claude-mem

# 或手动安装
git clone https://github.com/thedotmack/claude-mem
cd claude-mem && bun install
```

## 性能

- **Token 节省**: ~10x (通过智能上下文过滤)
- **Web 界面**: `http://localhost:37777` 实时监控
- 无标准基准 (LongMemEval/LoCoMo) 数据

## 特殊功能

- **Endless Mode** (Beta): 仿生记忆架构，支持超长会话
- **Law Study Mode**: 法律学习专用配置
- **Web 监控界面**: 实时浏览和搜索历史

## 优势

- **39.7k Stars**: Claude Code 生态中最受欢迎的记忆方案
- **零配置**: 安装插件即用，6 个 Hook 全自动
- **AI 压缩**: 不是简单存储，有语义理解的压缩
- **混合检索**: 关键词 + 语义，准确率高
- **全本地**: 数据不离开机器
- **活跃社区**: 532 个已合并 PR，74 个开放 PR

## 劣势

- **AGPL 许可**: 网络部署需开源，商用受限
- **Token 消耗**: 密集使用时消耗较多 API token
- **安全问题**: HTTP API (37777端口) 无认证，本地进程可访问会话数据
- **依赖 Bun**: 需要 Bun 运行时
- **仅限 Claude Code**: 不支持 Codex CLI、Cursor 等其他 Agent

## 与其他方案对比

| 对比维度 | Claude-Mem | Claude Code 原生 | Letta |
|---------|-----------|----------------|-------|
| 安装方式 | 插件安装 | 内置 | 独立CLI |
| 自动捕获 | ✅ 6个Hook | ✅ auto-memory | ❌ 需主动调用 |
| 语义检索 | ✅ Chroma | ❌ | ✅ |
| AI 压缩 | ✅ | ❌ | ✅ |
| 跨CLI支持 | ❌ Claude Only | ❌ Claude Only | ✅ 多CLI |
| Stars | 39.7k | N/A | 21.6k |

## 来源

- [GitHub 仓库](https://github.com/thedotmack/claude-mem)
- [GitHub Discussions](https://github.com/thedotmack/claude-mem/discussions)
