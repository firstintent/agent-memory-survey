# 基准测试汇总

## 主要基准测试

### LongMemEval

长期记忆评估基准，测试 Agent 跨多轮对话的记忆保持能力。

| 排名 | 系统 | 得分 | 备注 |
|------|------|------|------|
| 1 | **Supermemory** | 99% | LLM 压缩方案, 2026.03 |
| 2 | **Hindsight** | 91.4% | 首个超 90% 的系统 |
| 3 | **RetainDB** | 88%* | preference recall, 2026.03 新 |
| 4 | Zep | 71% | 时序知识图谱 |
| 5 | Mem0 | 49% | 向量+图混合 |

*RetainDB 的 88% 是 preference recall 指标，与其他系统的评估维度可能不完全一致。

### LoCoMo (Long-term Conversational Memory)

Snap Research 提出，评估长对话记忆的多个维度。

| 系统 | F1 改进 | 备注 |
|------|---------|------|
| **Memvid** | +35% SOTA | 单文件方案 |
| **SimpleMem** | +26.4% | 语义无损压缩 |
| SimpleMem | 30x token 节省 | |
| Memvid | +76% 多跳推理 | |

### MemoryAgentBench

测试 4 项核心能力：

1. **精确检索** (Accurate Retrieval) — 能否准确找到特定信息
2. **测试时学习** (Test-time Learning) — 能否从新信息快速学习
3. **长程理解** (Long-range Understanding) — 跨长距离的一致理解
4. **冲突消解** (Conflict Resolution) — 处理矛盾信息的能力

### AMA-Bench

评估长周期 Agent 记忆，关注多会话推理。

## 性能指标对比

| 系统 | LongMemEval | LoCoMo | 检索延迟 | Token 效率 |
|------|-------------|--------|---------|-----------|
| Supermemory | 99% | — | — | LLM 压缩 |
| Hindsight | 91.4% | — | — | 混合 |
| RetainDB | 88%* | — | — | — |
| Memvid | — | +35% | 0.025ms P50 | 单文件 |
| SimpleMem | — | +26.4% | — | 30x 节省 |
| SuperLocalMemory | — | 74.8% (local) | — | 60% zero-LLM |
| Zep | 71% | — | — | — |
| Mem0 | 49% | — | — | — |

## 延迟基准

| 系统 | P50 | P99 | 备注 |
|------|-----|-----|------|
| **Memvid** | 0.025ms | 0.075ms | Rust 重写后 |
| Redis (向量) | <1ms | <5ms | 内存数据库 |
| ChromaDB | ~10ms | ~50ms | 本地部署 |
| Qdrant | ~5ms | ~20ms | HNSW 索引 |

## 关键发现

1. **LLM-Native 方案碾压向量方案**：Supermemory (99%) vs Mem0 (49%)，差距 50 个百分点
2. **时序推理是分水岭**：Hindsight (91.4%) vs Zep (71%)，TEMPR 架构显著优于简单时序图谱
3. **单文件方案出人意料**：Memvid 在 LoCoMo 上 +35%，证明不需要复杂基础设施
4. **Token 效率差异巨大**：SimpleMem 30x 节省，SuperLocalMemory 60% 操作不需要 LLM

## 基准测试的局限性

- 各基准测试维度不同，跨基准对比需谨慎
- 大多数基准测试单 Agent 场景，多 Agent 评估缺失
- 实验室基准 ≠ 生产环境表现
- 新系统的基准可能是自己跑的，需要第三方复现
