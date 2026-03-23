# 自研记忆系统建议

> 基于 30+ 个项目的调研，给出 2026.03 自研 Agent 记忆系统的架构建议。

## 一、核心结论

### 不要做什么

| ❌ 避免 | 原因 | 数据支撑 |
|---------|------|---------|
| 纯向量方案 | 已被全面超越 | Mem0 49% vs Supermemory 99% |
| 自建向量DB | 基础设施负担重，收益低 | Google Always-On 用 SQLite 就够了 |
| 抄 Claude Code 原生记忆 | 无语义检索，纯文件方案天花板低 | 所有 Agent 自带记忆都缺语义检索 |
| 做纯 SaaS | AGPL/隐私是痛点，本地优先是趋势 | SuperLocalMemory 74.8% 纯本地 |

### 应该做什么

| ✅ 做 | 原因 | 参考 |
|-------|------|------|
| LLM-Native 压缩 | 99% 准确率，无向量开销 | Supermemory |
| 时间推理 + 冲突消解 | 真正的"记忆"必须处理时间 | Hindsight TEMPR |
| Hook 自动捕获 | 零手动成本是爆火前提 | Claude-Mem 39.7k stars |
| MCP 协议接入 | Claude Code/Codex CLI 原生支持 | Letta, SuperLocalMemory |
| 混合检索 | 关键词 + 语义 > 纯语义 | Claude-Mem, Memvid |

## 二、推荐架构

```
┌─────────────────────────────────────────────────────┐
│                    接入层                             │
│  MCP Server + CLI Hooks + REST API                   │
│  (Claude Code / Codex CLI / Cursor 原生接入)          │
├─────────────────────────────────────────────────────┤
│                   捕获层                              │
│  自动捕获 (6+ Hook)     +     主动记忆 (/remember)    │
│  ← Claude-Mem 模式              ← Letta 模式         │
├─────────────────────────────────────────────────────┤
│                   处理层                              │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐          │
│  │ LLM 压缩  │  │ 实体抽取  │  │ 冲突消解   │          │
│  │Supermemory│  │ Cognee式  │  │Hindsight式│          │
│  └──────────┘  └──────────┘  └───────────┘          │
├─────────────────────────────────────────────────────┤
│                   检索层                              │
│  混合检索: BM25 关键词 + 语义嵌入 + 时间加权           │
│  三层查询: 索引 → 上下文 → 详细  (Claude-Mem 模式)     │
├─────────────────────────────────────────────────────┤
│                   存储层                              │
│  SQLite (结构化) + 轻量向量索引 (本地嵌入)             │
│  全本地，零外部依赖，可 git 管理                       │
└─────────────────────────────────────────────────────┘
```

## 三、五个核心模块详解

### 模块 1: 自动捕获 (参考 Claude-Mem)

**为什么最重要**: Claude-Mem 39.7k Stars 证明了"零手动"是用户最看重的。

```
会话开始 → [Hook] 注入相关历史上下文
用户提问 → [Hook] 捕获用户意图
工具调用 → [Hook] 捕获工具观测和结果
会话结束 → [Hook] 生成会话摘要
空闲时   → [后台] 压缩、整合、清理
```

**关键设计**:
- Hook 必须轻量（<100ms），不能拖慢主流程
- 捕获原始数据，压缩异步进行
- 智能过滤：不是所有工具调用都值得记住

### 模块 2: LLM-Native 压缩 (参考 Supermemory)

**为什么**: 99% LongMemEval 证明 LLM 压缩 >> 向量嵌入。

```python
# 伪代码：LLM 压缩管线
def compress(raw_observations: list[str]) -> Memory:
    # Step 1: 提取关键事实
    facts = llm.extract_facts(raw_observations)

    # Step 2: 去重 + 合并
    merged = llm.merge_with_existing(facts, existing_memories)

    # Step 3: 冲突消解 (新事实覆盖旧事实)
    resolved = llm.resolve_conflicts(merged)

    # Step 4: 生成压缩摘要
    summary = llm.compress(resolved)

    return Memory(facts=resolved, summary=summary, ts=now())
```

**关键设计**:
- 压缩在后台异步进行，不阻塞用户
- 使用轻量模型（Haiku/Flash-Lite）压缩，降低成本
- 保留原始数据 + 压缩版本，检索时按需展开

### 模块 3: 时间推理 (参考 Hindsight TEMPR)

**为什么**: 91.4% SOTA，是区分"数据库"和"记忆"的关键。

```
三个操作:
  retain(fact, context, ts)  → 保留新记忆
  recall(query, time_range)  → 时间感知检索
  reflect()                  → 后台整合和反思

四个记忆网络:
  world_facts     → "项目使用 PostgreSQL"
  experiences     → "3/15 修了登录 bug，原因是..."
  entity_summary  → "用户 Rob: 偏好 TS, 熟悉 Rust"
  beliefs         → "该项目的测试覆盖率在提升" (可演化)
```

**关键设计**:
- 每条记忆带时间戳和有效期
- 冲突消解规则: 新事实默认覆盖旧事实，但保留历史
- reflect() 定期整合，合并碎片记忆，更新信念

### 模块 4: 混合检索 (参考 Claude-Mem + Memvid)

**为什么**: 纯语义检索会"相似但不相关"，混合检索更精准。

```
查询 → 并行执行:
  ├── BM25 关键词匹配 (精确)
  ├── 语义嵌入匹配 (模糊)
  └── 时间加权 (最近优先)
      ↓
  加权融合排序
      ↓
  Top-K 结果 → LLM 判断相关性 → 返回
```

**关键设计**:
- BM25 用于精确匹配（函数名、变量名、错误码）
- 语义用于模糊匹配（"上次讨论的架构"）
- 时间加权: 最近的记忆权重更高
- 最后一步用 LLM 判断，过滤假阳性
- 本地嵌入模型（不依赖远程 API）

### 模块 5: 存储 (参考 Codex CLI + Google Always-On)

**为什么**: 零外部依赖是本地工具的生命线。

```
SQLite 表结构:
  memories:     id, content, summary, entities, ts, expires_at, importance
  facts:        id, subject, predicate, object, valid_from, valid_to
  sessions:     id, start_ts, end_ts, summary, tool_count
  observations: id, session_id, hook_type, raw_content, ts

本地嵌入索引:
  embeddings:   id, memory_id, vector (本地模型生成)

文件导出:
  ~/.my-memory/memory.db          # SQLite 主库
  ~/.my-memory/exports/MEMORY.md  # 可读导出 (可 git)
  ~/.my-memory/config.json        # 配置
```

**关键设计**:
- SQLite 单文件，零依赖
- 本地嵌入模型（gte-small 或类似），不需要远程 API
- 定期导出为 Markdown，可版本控制
- 自动清理过期记忆，控制存储增长

## 四、与现有方案的差异化

| 维度 | 现有最佳 | 自研目标 | 差异化点 |
|------|---------|---------|---------|
| 准确率 | Supermemory 99% | 接近 | LLM 压缩 + 时间推理 |
| CLI集成 | Letta (原生) | 更好 | MCP + Hook 双通道 |
| 自动化 | Claude-Mem (Hook) | 同级 | 更智能的过滤 |
| 时间推理 | Hindsight 91.4% | 同级 | TEMPR 简化版 |
| 隐私 | SuperLocalMemory | 同级 | 纯本地 SQLite |
| 跨CLI | memory-bridge | 原生 | 一份记忆多CLI共享 |

**真正的差异化**: 目前没有一个系统同时具备以上所有能力。

## 五、MVP 路线图

### Phase 1: 最小可用 (1-2天 CC时间)

```
✅ SQLite 存储 + MCP Server
✅ 基础 /remember 和 /recall 工具
✅ BM25 关键词检索
✅ Claude Code Hook: session_start + session_end
```

交付物: 能在 Claude Code 中用 MCP 存取记忆。

### Phase 2: 智能化 (2-3天)

```
✅ LLM 压缩 (后台异步)
✅ 6 个 Hook 全量捕获
✅ 本地嵌入 + 混合检索
✅ 自动注入相关上下文
```

交付物: 接近 Claude-Mem 的自动化体验。

### Phase 3: 时间推理 (2-3天)

```
✅ 实体抽取 + 事实存储
✅ 冲突消解 (新覆盖旧)
✅ 时间加权检索
✅ reflect() 后台整合
```

交付物: 接近 Hindsight 的时间感知能力。

### Phase 4: 跨CLI + 生态 (1-2天)

```
✅ Codex CLI 支持
✅ Cursor 支持 (通过 MCP)
✅ Web 监控界面
✅ 记忆导出/导入
```

交付物: 完整的跨 CLI 记忆系统。

**总计预估**: 6-10 天 CC+gstack 时间 (人工约 2-3 个月)

## 六、技术选型建议

| 组件 | 推荐 | 原因 |
|------|------|------|
| 语言 | TypeScript (Bun) | Claude Code 生态主流, Hook 脚本自然 |
| 存储 | SQLite (better-sqlite3) | 零依赖, 单文件, 性能好 |
| 嵌入 | gte-small 本地模型 | 不依赖远程 API |
| 压缩 LLM | Claude Haiku / Gemini Flash | 成本低, 速度快 |
| 接口 | MCP Server | Claude Code/Codex CLI 原生 |
| 检索 | 自建 BM25 + 向量 | 轻量, 不需要 Chroma/Qdrant |
| 许可 | MIT | 最大灵活度 |

## 七、风险和注意事项

1. **Token 成本**: LLM 压缩和检索都需要 API 调用，密集使用时成本可能高。建议用 Haiku/Flash-Lite 做压缩。
2. **安全**: Claude-Mem 被爆 HTTP API 无认证。自研时 MCP 走 stdio，避免开放端口。
3. **AGPL 污染**: 如果参考 Claude-Mem 代码，注意 AGPL 传染。建议只参考架构思路，从零实现。
4. **过度记忆**: 不是所有信息都值得记住。需要智能过滤 + 自动遗忘机制。
5. **冷启动**: 新项目没有历史记忆，体验差。可以支持从 CLAUDE.md / .cursor/rules 导入初始上下文。
