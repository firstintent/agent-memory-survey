# Agent 自带记忆系统横向对比

## 对比总表

| Agent | 存储方式 | 记忆类型 | 持久范围 | 用户控制 | 自动记忆 | 语义检索 | 版本控制 |
|-------|---------|---------|---------|---------|---------|---------|---------|
| **Claude Code** | 文件 (CLAUDE.md + memory/) | 规则 + 自动记忆 | 跨会话 | ✅ 完全 | ✅ | ❌ | ✅ |
| **Cursor** | 文件 (.cursor/rules/) | 规则 + Memory Banks | 跨会话 | ✅ 完全 | ❌ | ❌ | ✅ |
| **GitHub Copilot** | 云端 (repo-scoped) | 仓库模式 | 跨会话 | ⚠️ 有限 | ✅ | ❌ | ❌ |
| **Windsurf** | 文件 + 云端 | 规则 + Cascade 记忆 | 跨会话 | ✅ 完全 | ✅ | ❌ | ⚠️ |
| **Aider** | 向量 DB (LanceDB) | 向量嵌入 | 跨会话 | ⚠️ 部分 | ✅ | ✅ | ❌ |
| **Devin** | 应用 KB + 文件 | 知识库 + Playbooks | 跨会话 | ✅ 完全 | ✅ | ❌ | ❌ |
| **Codex CLI** | SQLite + 文件 | 结构化记忆 + 技能 | 跨会话 | ✅ 完全 | ✅ | ❌ | ✅ |
| **ChatGPT** | 云端 (user-scoped) | 保存的记忆 | 跨会话 | ✅ 完全 | ✅ | ❌ | ❌ |
| **Gemini** | 云端 (user-scoped) | 个人智能 + 保存信息 | 跨会话 | ⚠️ 部分 | ✅ | ❌ | ❌ |
| **Kiro** | 文件 (.amazonq/rules/) | Memory Banks | 跨会话 | ✅ 完全 | ✅ | ❌ | ✅ |
| **Augment Code** | 应用记忆 | 偏好 + 决策 | 跨会话 | ✅ 完全 (Review) | ✅ | ❌ | ❌ |
| **Cline** | 项目文件 (Markdown) | 结构化文档 | 跨会话 | ✅ 完全 | ❌ | ❌ | ✅ |
| **Replit Agent** | 内存 (压缩) | 轨迹压缩 | 仅会话内 | ⚠️ 有限 | ✅ | ❌ | ❌ |

## 六种记忆模式

### 模式 1: 文件持久化
**代表**: Claude Code, Cursor, Cline, Kiro

```
项目根目录/
├── CLAUDE.md / .cursorules / .clinerules    # 手动规则
├── .claude/rules/ / .cursor/rules/          # 分类规则
└── memory/                                   # 自动记忆 (Claude Code)
```

**优势**: 可版本控制、透明、用户完全可控
**劣势**: 无语义检索、容量受 context window 限制

### 模式 2: 云端记忆
**代表**: GitHub Copilot, ChatGPT, Gemini

**特点**: 账户/仓库级存储，跨设备同步
**优势**: 零配置、跨设备
**劣势**: 隐私风险、不可版本控制、有限的用户控制

### 模式 3: 向量嵌入
**代表**: Aider (LanceDB)

**特点**: 本地向量数据库，语义相似搜索
**优势**: 支持语义检索
**劣势**: 依赖 embedding 模型、不透明

### 模式 4: 混合方式 (规则 + 自动)
**代表**: Windsurf, Augment Code

**特点**: 手动规则 + AI 自动生成记忆
**Windsurf**: Rules (手动) → Memories (Cascade 自动生成) → 分层加载
**Augment**: Memory Review — AI 提议，用户审批

### 模式 5: 知识库
**代表**: Devin

**特点**: 应用内结构化知识库 + 自动 Playbooks
**优势**: 结构化、可复用
**劣势**: 绑定特定平台

### 模式 6: 结构化存储
**代表**: Codex CLI

**特点**: SQLite 提取 + Markdown 整合的两阶段管线
**Phase 1**: 从完成的任务中提取结构化记忆到 SQLite
**Phase 2**: 整合更新 MEMORY.md
**优势**: 结构化 + 可读性
**劣势**: 两步流程可能有延迟

## 记忆能力雷达图

```
                    自动记忆
                      ★★★★★
                     ╱       ╲
           版本控制 ★★★★★     ★★★★★ 用户控制
                  ╱               ╲
        语义检索 ★☆☆☆☆ ——— ★★★☆☆ 跨设备
                  ╲               ╱
        隐私保护 ★★★★☆     ★★☆☆☆ 团队共享
                     ╲       ╱
                      ★★★☆☆
                    容量/扩展性
```

**Claude Code** 在版本控制、用户控制、隐私保护上最强，但缺少语义检索。
**Aider** 是唯一有语义检索的，但用户控制较弱。
**Codex CLI** 在结构化存储上最优，但刚起步。

## 对自研的启示

1. **文件 + 数据库混合** (Codex CLI 模式) 是最平衡的方案
2. **Memory Review** (Augment Code) 模式值得借鉴：AI 提议，用户审批
3. **CLAUDE.md 的分层** (全局 → 项目 → 目录) 是好的组织模式
4. **缺失的能力**: 所有现有 Agent 都缺少高质量语义检索 (除 Aider)
5. **机会**: 将 Supermemory/Hindsight 级别的记忆集成到 CLI Agent 中
