# Mem0

## Project Info

| Field | Value |
|-------|-------|
| **GitHub** | [mem0ai/mem0](https://github.com/mem0ai/mem0) |
| **Stars** | 50.7k |
| **License** | Apache 2.0 |
| **Language** | Python (63%), TypeScript (26%) |
| **Status** | Active (v1.0.0 released), SaaS + self-hosted |
| **Paper** | [arXiv:2504.19413](https://arxiv.org/abs/2504.19413) |

## Summary

Mem0 is a universal memory layer for AI agents that extracts, consolidates, and retrieves salient information from conversations. It uses a hybrid storage approach combining vector databases, key-value stores, and graph databases to store different types of information optimally. The platform offers both a managed SaaS product (mem0.ai) and a fully self-hostable open-source version.

## Architecture

```
User Message
    |
    v
  add() --> Fact/Preference Extraction (LLM)
    |
    +--> Vector DB (semantic embeddings)
    +--> Key-Value DB (structured facts)
    +--> Graph DB (entity relationships)
    |
  search() --> Hybrid Retrieval --> Ranked Results
```

When `add()` is called, the system uses an LLM to extract relevant facts and preferences, then distributes them across three storage tiers. On retrieval via `search()`, results are merged from all stores for comprehensive context.

The graph memory component tracks complex entity relationships (e.g., "Alice works at Acme" + "Alice moved to NYC" = connected entity graph), enabling multi-hop reasoning that pure vector search misses.

## Memory Types

| Type | Description | Storage |
|------|-------------|---------|
| **Semantic** | Dense vector embeddings of extracted facts | Vector DB |
| **Structured** | Key-value pairs of user preferences/facts | KV Store |
| **Relational** | Entity-relationship graphs | Graph DB |

Memories are automatically deduplicated, merged, and updated as new information arrives. Contradictions are resolved by preferring newer information.

## Storage Backends

**Vector Databases:**
- Qdrant
- Chroma
- pgvector (PostgreSQL)
- Weaviate
- Milvus
- Amazon ElastiCache for Valkey

**Graph Databases:**
- Neo4j
- FalkorDB
- KuzuDB
- Amazon Neptune Analytics

**Key-Value:**
- Redis
- In-memory (development)

## Retrieval Methods

1. **Semantic Search** -- Cosine similarity over dense embeddings
2. **Graph Traversal** -- Multi-hop entity relationship queries
3. **Keyword/Metadata Filtering** -- Structured attribute matching
4. **Hybrid Fusion** -- Combined scoring across all backends

Graph memory adds approximately 2% overall score improvement over base vector-only configuration.

## Agent Integration

- **LangChain/LangGraph**: First-class support; build customer bots with LangGraph + Mem0
- **CrewAI**: Tailor CrewAI agent outputs with personalized Mem0 memory
- **Vercel AI SDK**: JavaScript/TypeScript integration
- **AutoGen**: Multi-agent memory sharing
- **Custom**: REST API and Python/TypeScript SDKs

```python
from mem0 import Memory

m = Memory()
m.add("I prefer dark mode and use Python 3.12", user_id="alice")
results = m.search("What are Alice's preferences?", user_id="alice")
# [{'memory': 'Prefers dark mode', 'score': 0.95}, ...]
```

## Benchmarks

| Benchmark | Score | Notes |
|-----------|-------|-------|
| **LongMemEval** | ~49% | Lagging behind newer systems like Zep (71%) |
| **LLM-as-Judge** | 26% relative improvement over OpenAI | Mem0 research metric |
| **Latency** | 91% lower p95 vs baseline | Production deployment |
| **Token Cost** | 90%+ savings | Compared to full-context approaches |

## Strengths

- Massive community (50k+ stars), well-documented, production-tested
- Hybrid vector + graph architecture covers both semantic and relational queries
- Extensive storage backend options (5+ vector DBs, 4+ graph DBs)
- First-class integrations with major agent frameworks
- SaaS option for teams that do not want to self-host
- AWS partnership with Neptune + ElastiCache reference architecture

## Weaknesses

- 49% LongMemEval accuracy lags significantly behind newer systems (Zep at 71%, Memvid at 35% SOTA improvement)
- Graph memory adds only marginal improvement (~2%) over vector-only
- Complex deployment with multiple database dependencies for full hybrid setup
- SaaS dependency risk for production users
- Memory extraction quality depends heavily on underlying LLM
- No built-in temporal reasoning (timestamps exist but no temporal graph)

## Usage Example

```python
from mem0 import Memory

config = {
    "graph_store": {
        "provider": "neo4j",
        "config": {
            "url": "neo4j://localhost:7687",
            "username": "neo4j",
            "password": "password"
        }
    },
    "vector_store": {
        "provider": "qdrant",
        "config": {
            "host": "localhost",
            "port": 6333
        }
    }
}

m = Memory.from_config(config)

# Add memories from a conversation
m.add(
    "I'm moving to San Francisco next month. I work at OpenAI as a researcher.",
    user_id="user_123"
)

# Search memories
results = m.search("Where does the user work?", user_id="user_123")
print(results)
# [{'memory': 'Works at OpenAI as a researcher', 'score': 0.94}]

# Graph queries for relationships
m.search("What entities are connected to OpenAI?", user_id="user_123")
```
