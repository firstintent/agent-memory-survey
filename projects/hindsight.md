# Hindsight by Vectorize

- **GitHub**: https://github.com/vectorize-io/hindsight
- **Documentation**: https://hindsight.vectorize.io/
- **License**: MIT
- **Paper**: [arXiv:2512.12818](https://arxiv.org/abs/2512.12818)
- **Developed by**: Vectorize.io, in collaboration with Virginia Tech and The Washington Post

## Summary

Hindsight is an open-source agent memory system that organizes AI memory into four biologically-inspired networks and provides three core operations (retain, recall, reflect) to give agents human-like long-term memory. It was the first system to break 90% on the LongMemEval benchmark, achieving 91.4% accuracy -- far exceeding prior approaches based on vanilla RAG. Hindsight's dual architecture (TEMPR for retrieval, CARA for reasoning) treats memory as a structured, first-class substrate for reasoning rather than an external retrieval layer that dumps text chunks into prompts. The March 2026 launch post on X ("killed RAG") drew 433 likes and 29k views, and the project has been widely recommended by Claude Code users for persistent agent memory.

## Architecture Overview

Hindsight is built on two core architectural components:

### TEMPR (Temporal Entity Memory Priming Retrieval)

TEMPR handles memory retention and recall by running four parallel search strategies:

1. **Semantic vector similarity** -- dense embedding search
2. **Keyword matching via BM25** -- sparse lexical retrieval
3. **Graph traversal** -- walks shared entity connections
4. **Temporal filtering** -- constrains results by time windows

Results from all four channels are merged using **Reciprocal Rank Fusion (RRF)** and passed through a neural reranker for final precision. This hybrid approach ensures that queries combining semantic meaning, specific terms, entity relationships, and time constraints all resolve accurately.

### CARA (Coherent Adaptive Reasoning Agents)

CARA handles preference-aware reflection by integrating configurable disposition parameters into the reasoning process:

- **Skepticism** -- how critically the agent evaluates incoming information
- **Literalism** -- how literally vs. figuratively the agent interprets statements
- **Empathy** -- how much weight the agent gives to emotional/social context

These parameters address a key problem: inconsistent reasoning across sessions. CARA ensures that an agent's personality and judgment style persist reliably.

### Retain Pipeline Internals

When `retain()` is called, an LLM extracts:
- Key facts and assertions
- Temporal data (timestamps, date ranges)
- Named entities and relationships
- Sentiment and confidence scores

These are normalized into canonical entities, time series, and search indexes with metadata, creating the pathways used by recall and reflect.

## Memory Types Supported

Hindsight organizes memory into **four separate networks**:

| Network | Purpose | Example |
|---------|---------|---------|
| **World** | Facts about the external world | "Python 3.12 was released in October 2023" |
| **Experiences** | The agent's own interaction history | "User asked me to refactor the auth module on March 5" |
| **Entity Summaries** | Synthesized profiles of people, projects, concepts | "Alice: senior engineer at Google, promoted June 2025" |
| **Evolving Beliefs** | Opinions and beliefs with confidence scores that update over time | "This codebase likely uses a hexagonal architecture (confidence: 0.8)" |

This separation prevents fact contamination -- the agent won't confuse its own experiences with objective world knowledge, and beliefs are tracked with explicit confidence rather than being asserted as facts.

## Storage Backends

- **Built-in server**: Hindsight runs as a standalone service (default port 8888)
- **Local with Ollama**: Can run entirely locally with no API keys using Ollama for embeddings and LLM inference
- **Vectorize Cloud**: Managed hosted option via Vectorize.io
- **Underlying storage**: Uses vector indexes (for embeddings), BM25 indexes (for keyword search), and a graph store (for entity relationships)

## Retrieval Methods

| Method | Description |
|--------|-------------|
| **Vector similarity** | Dense semantic search over embeddings |
| **BM25** | Sparse keyword/term matching |
| **Graph traversal** | Follow entity relationship edges |
| **Temporal filtering** | Time-windowed queries |
| **Reciprocal Rank Fusion** | Merges results from all four methods |
| **Neural reranking** | Final precision pass on merged results |
| **Reflect** | LLM-powered synthesis that generates new observations and insights from existing memories |

## Agent/CLI Integration

### Claude Code
Hindsight is frequently recommended by Claude Code users for adding persistent memory. Integration is typically done via the MCP (Model Context Protocol) server or direct Python client calls in hooks/skills.

### Framework Integrations
- **Pydantic AI**: [5-line integration](https://hindsight.vectorize.io/blog/2026/03/09/pydantic-ai-persistent-memory)
- **Microsoft Agent Framework**: Documented integration pattern
- **Hermes (self-improving agent)**: Official blog post on memory upgrade
- **Generic**: Any agent that can make HTTP calls or use the Python SDK

### Python SDK Installation

```bash
pip install hindsight-client -U
```

## Performance Benchmarks

| Benchmark | Score | Notes |
|-----------|-------|-------|
| **LongMemEval** | **91.4%** | First system to break 90%; prior SOTA was significantly lower |
| vs. Mem0 | ~49% | Hindsight nearly doubles Mem0's score |
| vs. Zep | ~71% | Hindsight exceeds Zep by ~20 points |

The benchmark measures long-term memory across multi-session conversations, temporal reasoning, knowledge conflict resolution, and entity tracking -- all areas where traditional RAG struggles.

## Strengths

- **Best-in-class benchmark performance** at 91.4% LongMemEval (at time of release)
- **Four-network separation** prevents memory contamination between facts, experiences, beliefs, and entity summaries
- **Hybrid retrieval** (vector + BM25 + graph + temporal) handles diverse query types that no single method covers
- **Temporal awareness** is first-class, not bolted on -- critical for "what changed" and "when did" queries
- **Evolving beliefs with confidence** let agents update their understanding gracefully
- **MIT license** with no usage restrictions
- **Runs locally** with Ollama, no cloud dependency required
- **Active development** with cookbook, blog posts, and framework integrations shipping regularly in 2026

## Weaknesses

- **Requires running a separate server** -- not an embeddable library, adds operational complexity
- **LLM dependency for retain** -- every memory insertion requires an LLM call for extraction, which adds latency and cost
- **Relatively new** (December 2025 paper, early 2026 releases) -- less battle-tested in production than older systems
- **CARA disposition tuning** requires manual configuration; no automatic calibration
- **Documentation is solid but ecosystem is still growing** -- fewer third-party integrations than Letta or Mem0
- **No built-in Claude Code plugin** -- integration requires manual setup via SDK or MCP, unlike Supermemory's one-command install

## Key Code Example

```python
from hindsight_client import Hindsight

# Connect to local Hindsight server
client = Hindsight(base_url="http://localhost:8888")

# Retain memories with temporal context
client.retain(
    bank_id="project-alpha",
    content="Alice joined the team as a senior engineer",
    context="team update",
    timestamp="2025-09-01T09:00:00Z"
)

client.retain(
    bank_id="project-alpha",
    content="Alice was promoted to tech lead after shipping the v2 release",
    context="career update",
    timestamp="2026-01-15T10:00:00Z"
)

# Recall -- hybrid search across all four retrieval methods
results = client.recall(
    bank_id="project-alpha",
    query="What is Alice's current role?"
)
for r in results:
    print(f"[{r.score:.2f}] {r.text}")

# Reflect -- LLM-synthesized answer grounded in memories
answer = client.reflect(
    bank_id="project-alpha",
    query="Summarize Alice's trajectory on the team"
)
print(answer.text)
# Output: "Alice joined project-alpha as a senior engineer in September 2025
#          and was promoted to tech lead in January 2026 after shipping the v2 release."
```

---

*Last updated: March 2026*
