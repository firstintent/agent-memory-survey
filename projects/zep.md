# Zep

## Project Info

| Field | Value |
|-------|-------|
| **GitHub** | [getzep/graphiti](https://github.com/getzep/graphiti) (core engine) |
| **Stars** | 24.1k (Graphiti) |
| **License** | Open Source |
| **Language** | Python |
| **Status** | Active, commercial product + open-source Graphiti engine |
| **Paper** | [arXiv:2501.13956](https://arxiv.org/abs/2501.13956) (January 2025) |
| **Website** | [getzep.com](https://www.getzep.com/) |

## Summary

Zep is a context engineering and agent memory platform built around Graphiti, a temporally-aware knowledge graph engine. Unlike vector-only approaches, Zep maintains a temporal knowledge graph that tracks how entities and relationships change over time. Facts have validity windows -- when information changes, old facts are invalidated but not deleted, preserving historical context. This makes Zep particularly strong at enterprise tasks requiring cross-session information synthesis and temporal reasoning.

## Architecture

```
Incoming Data (conversations, business data)
    |
    v
  Graphiti Engine
    |
    +--> Episode Subgraph (raw data provenance)
    +--> Semantic Entity Subgraph (entities + relationships)
    +--> Community Subgraph (cluster summaries)
    |
  Hybrid Retrieval (semantic + BM25 + graph traversal)
    |
    v
  Temporally-Aware Results with Provenance
```

Graphiti uses a three-tier hierarchical subgraph model:

1. **Episode Subgraph** -- Raw source data (conversations, documents) with timestamps. Every entity and relationship traces back to the episodes that produced it.
2. **Semantic Entity Subgraph** -- Extracted entities and typed relationships with temporal validity windows. When facts change, old edges are invalidated with end timestamps.
3. **Community Subgraph** -- Automatically generated cluster summaries for high-level reasoning.

Graph construction is incremental: new data integrates immediately without batch recomputation.

## Memory Types

| Type | Description | Temporal |
|------|-------------|----------|
| **Episodic** | Raw conversation/document records | Timestamped |
| **Entity** | Named entities with attributes | Validity windows |
| **Relational** | Typed relationships between entities | Validity windows |
| **Community** | Cluster-level summaries | Auto-updated |

The temporal dimension is the key differentiator. Each fact carries `valid_from` and `valid_to` timestamps, enabling queries like "Where did Alice work in 2024?" even after her employment has changed.

## Storage Backends

| Backend | Version | Notes |
|---------|---------|-------|
| **Neo4j** | 5.26+ | Primary/default |
| **FalkorDB** | 1.1.2+ | Lightweight alternative |
| **KuzuDB** | 0.11.2+ | Embedded option |
| **Amazon Neptune** | Database Cluster | Enterprise |
| **Amazon Neptune Analytics** | + OpenSearch Serverless | Full-text search |

## Retrieval Methods

1. **Semantic Embeddings** -- Dense vector similarity for conceptual matching
2. **BM25 Keyword Search** -- Exact term matching for precision
3. **Graph Traversal** -- Multi-hop relationship following with temporal filters
4. **Hybrid Fusion** -- Combined scoring across all three channels

All retrieval is low-latency by design, with 90% latency reduction compared to full-context baselines.

## Agent Integration

- **MCP Server** -- Model Context Protocol integration for AI assistants
- **REST API** -- FastAPI-based service for programmatic access
- **Python SDK** -- Direct library integration
- **LLM Support** -- OpenAI, Anthropic, Groq, Google Gemini with structured output
- **Flexible Ontology** -- Pydantic models for prescribed entity/relationship types, or auto-learned types

```python
from graphiti_core import Graphiti

graphiti = Graphiti(neo4j_driver, llm_client)

# Add episodic data
await graphiti.add_episode(
    name="meeting_notes",
    episode_body="Alice joined Acme Corp as VP Engineering in March 2025.",
    source_description="meeting transcript"
)

# Temporal query
results = await graphiti.search("Who was VP Engineering at Acme in 2025?")
```

## Benchmarks

| Benchmark | Score | Comparison |
|-----------|-------|------------|
| **DMR (Deep Memory Retrieval)** | 94.8% | vs. MemGPT 93.4% |
| **LongMemEval** | ~71% | +18.5% over baseline |
| **Latency** | 90% reduction | vs. full-context baseline |

The 71% LongMemEval score serves as an important "old era" reference point -- substantially better than Mem0's 49% but now surpassed by newer systems like Memvid and SuperLocalMemory.

## Strengths

- Temporal knowledge graph is a genuine architectural innovation; no other system tracks fact validity windows as cleanly
- Strong benchmark results: 94.8% DMR, 71% LongMemEval
- Incremental graph construction (no batch recomputation)
- Full provenance tracking from entities back to source episodes
- Multiple graph database backends including embedded (KuzuDB)
- Well-documented research paper with clear methodology

## Weaknesses

- 71% LongMemEval now surpassed by newer systems (Memvid, SuperLocalMemory)
- Graph database dependency adds operational complexity
- Commercial product means some features may be gated
- Neo4j as primary backend has licensing implications for self-hosted deployments
- Community subgraph generation adds LLM cost overhead
- No built-in multimodal support

## Usage Example

```python
from graphiti_core import Graphiti
from graphiti_core.nodes import EpisodeType
from neo4j import AsyncGraphDatabase

driver = AsyncGraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "password"))
graphiti = Graphiti(driver, openai_client)

# Ingest a series of conversations over time
await graphiti.add_episode(
    name="jan_meeting",
    episode_body="Alice is the CTO of WidgetCo. They are launching Product X in Q2.",
    source_description="January board meeting",
    reference_time=datetime(2025, 1, 15)
)

await graphiti.add_episode(
    name="mar_update",
    episode_body="Alice left WidgetCo. Bob is now the interim CTO.",
    source_description="March status update",
    reference_time=datetime(2025, 3, 1)
)

# Temporal query: system knows Alice WAS CTO but Bob IS CTO
results = await graphiti.search("Who is the CTO of WidgetCo?")
# Returns: Bob (current), with Alice's prior role in history

# Historical query
results = await graphiti.search("Who was leading WidgetCo in January?")
# Returns: Alice (valid_from: Jan, valid_to: Mar)
```
