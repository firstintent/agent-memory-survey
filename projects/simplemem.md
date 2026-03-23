# SimpleMem

## Project Info

| Field | Value |
|-------|-------|
| **GitHub** | [aiming-lab/SimpleMem](https://github.com/aiming-lab/SimpleMem) |
| **Stars** | 3.2k |
| **License** | Open Source |
| **Language** | Python |
| **Status** | Active, research + pip-installable |
| **Paper** | [arXiv:2601.02553](https://arxiv.org/abs/2601.02553) |
| **PyPI** | `pip install simplemem` |

## Summary

SimpleMem is an efficient lifelong memory framework for LLM agents built around semantic lossless compression. It addresses the core tension in agent memory: full interaction histories are redundant and expensive, while aggressive summarization loses critical details. SimpleMem's three-stage pipeline achieves 30x token reduction while actually improving accuracy (+26.4% F1 on LoCoMo vs Mem0), proving that compression and quality are not mutually exclusive.

## Architecture

```
Raw Interaction Stream
    |
    v
  Stage 1: Semantic Structured Compression
    |  -- Resolve references (pronouns, etc.)
    |  -- Normalize timestamps to absolute
    |  -- Distill into atomic memory units
    |
    v
  Stage 2: Online Semantic Synthesis
    |  -- Intra-session deduplication
    |  -- Merge related fragments
    |  -- Implicit semantic density gating
    |
    v
  Stage 3: Intent-Aware Retrieval Planning
    |  -- Infer search intent from query
    |  -- Determine retrieval scope dynamically
    |  -- Parallel multi-view retrieval
    |
    v
  LanceDB (persistent vector storage)
    |
  Three-Index Retrieval
    +--> Semantic (1024-dim dense vectors)
    +--> Lexical (BM25 sparse keywords)
    +--> Symbolic (timestamps, entities, metadata)
```

### Stage Details

**Stage 1: Semantic Structured Compression** -- Converts unstructured dialogue into compact, atomic memory units. Each unit has resolved references (no dangling "he/she/it") and absolute timestamps. This is the "lossless" part: information density increases without semantic loss.

**Stage 2: Online Semantic Synthesis** -- Operates during the write phase within a session. When a new memory unit overlaps with existing ones, they are consolidated into a unified abstract representation. This eliminates redundancy before it reaches storage.

**Stage 3: Intent-Aware Retrieval Planning** -- At query time, the system infers what the user actually needs (not just keyword matching) and dynamically selects which indexes to query and how broadly to search.

## Memory Types

| Type | Description |
|------|-------------|
| **Atomic Memory Units** | Compressed, reference-resolved facts from interactions |
| **Synthesized Abstractions** | Merged representations of related memory units |
| **Multi-View Index Entries** | Semantic, lexical, and symbolic representations per unit |

## Storage Backends

| Component | Technology |
|-----------|------------|
| **Vector DB** | LanceDB (persistent, local) |
| **Semantic Index** | 1024-dimensional dense embeddings |
| **Lexical Index** | BM25 sparse keyword index |
| **Symbolic Index** | Structured metadata (timestamps, entities, persons) |

Docker deployment uses a local volume mount for LanceDB persistence.

## Retrieval Methods

All three indexes are queried in parallel, with ID-based deduplication to merge results:

1. **Semantic** -- Dense vector cosine similarity (conceptual matching)
2. **Lexical** -- BM25 sparse keyword search (exact term precision)
3. **Symbolic** -- Structured metadata filtering (temporal, entity-based)

The intent-aware planner decides how to weight each channel based on the query type.

## Agent Integration

- **Python SDK** -- `pip install simplemem`
- **Docker** -- Containerized deployment with persistent volumes
- **REST API** -- HTTP endpoints for language-agnostic integration

```python
from simplemem import SimpleMem

mem = SimpleMem()

# Add interaction memories
mem.add("User discussed moving to SF for a new role at OpenAI", session_id="s1")
mem.add("User mentioned they prefer morning meetings", session_id="s1")

# Retrieval with intent-aware planning
results = mem.search("Where is the user relocating?")
```

## Benchmarks

### LoCoMo-10 (GPT-4.1-mini)

| System | F1 Score | Retrieval Time | Total Time |
|--------|----------|----------------|------------|
| **SimpleMem** | **43.24%** | **388.3s** | **480.9s** |
| Mem0 | 34.20% | 583.4s | 1934.3s |
| LightMem | 24.63% | 577.1s | 675.9s |

### Cross-Session Performance

| System | Score |
|--------|-------|
| **SimpleMem** | **48.0** |
| Claude-Mem | 29.3 |

Key metrics:
- **+26.4% avg F1** improvement over Mem0
- **30x fewer inference tokens** through compression
- **50.2% faster retrieval** than baseline approaches
- **4x faster total time** than Mem0

## Strengths

- Semantic lossless compression is a principled approach with strong theoretical grounding
- 30x token reduction directly translates to cost savings at scale
- Three-index parallel retrieval covers semantic, lexical, and structured queries
- Intent-aware retrieval planning adapts to query type automatically
- Significant benchmark improvements over Mem0 (+26.4% F1)
- Simple pip install, no heavy database dependencies (LanceDB is embedded)
- Academic rigor with reproducible benchmarks

## Weaknesses

- Relatively small community (3.2k stars) compared to Mem0 (50.7k)
- LanceDB-only storage; no pluggable vector DB backends
- No graph database component for relational queries
- Compression quality depends on underlying LLM
- No built-in temporal reasoning
- No MCP server or IDE integration
- No multi-agent memory sharing features
- Research-stage project; production hardening unclear

## Usage Example

```python
from simplemem import SimpleMem

# Initialize with LanceDB storage
mem = SimpleMem(
    db_path="./memory_store",
    embedding_dim=1024
)

# Stage 1+2: Compression and synthesis happen automatically
mem.add(
    "During today's standup (Jan 15, 2025), Alice mentioned she's blocked on "
    "the API redesign. Bob offered to pair program tomorrow. The team agreed "
    "to push the deadline to next Friday.",
    session_id="standup_jan15"
)

# Atomic units created:
# - "Alice blocked on API redesign (2025-01-15)"
# - "Bob to pair program with Alice (2025-01-16)"
# - "API redesign deadline: 2025-01-24 (Friday)"

# Stage 3: Intent-aware retrieval
results = mem.search("What is blocking Alice?")
# Semantic index matches "blocked on API redesign"
# Symbolic index matches entity "Alice"
# Results fused and deduplicated

results = mem.search("When is the API deadline?")
# Symbolic index matches temporal "2025-01-24"
# Lexical index matches "deadline" + "API"
```
