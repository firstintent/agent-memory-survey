# A-MEM

## Project Info

| Field | Value |
|-------|-------|
| **GitHub** | [agiresearch/A-mem](https://github.com/agiresearch/A-mem) |
| **Stars** | 929 |
| **License** | MIT |
| **Language** | Python |
| **Status** | Active, NeurIPS 2025 paper |
| **Paper** | [arXiv:2502.12110](https://arxiv.org/abs/2502.12110) |
| **Venue** | NeurIPS 2025 (Poster) |

## Summary

A-MEM is an agentic memory system for LLM agents that dynamically organizes memories using principles from the Zettelkasten method. Unlike static memory stores that simply append and retrieve, A-MEM treats memory as a living, interconnected knowledge network. When new memories are added, the system analyzes existing memories to find connections, establishes meaningful links, and can trigger updates to the context and attributes of historical memories. This creates an evolving knowledge structure that improves over time.

## Architecture

```
New Memory Input
    |
    v
  LLM-Driven Note Generation
    |  -- Contextual description
    |  -- Keywords and tags
    |  -- Embedding generation
    |
    v
  Connection Analysis
    |  -- Query ChromaDB for similar memories
    |  -- Analyze relationships with LLM
    |  -- Establish bidirectional links
    |
    v
  Memory Evolution
    |  -- Update related memories' context
    |  -- Refine tags and descriptions
    |  -- Reorganize link structure
    |
    v
  ChromaDB (persistent vector storage)
    |
  Retrieval: embedding search + linked memory expansion
```

### Zettelkasten Principles

The Zettelkasten method, originally a paper-based note-taking system, provides three key principles adapted for agent memory:

1. **Atomic Notes** -- Each memory is a self-contained unit with clear boundaries
2. **Dynamic Indexing** -- Memories are indexed by content, keywords, and relationships rather than rigid categories
3. **Emergent Structure** -- The organization emerges from connections between notes rather than being imposed top-down

When a new memory is added:
1. A comprehensive note is generated with structured attributes (description, keywords, tags)
2. The system searches for related historical memories
3. Meaningful links are established where similarities exist
4. Connected memories may have their attributes updated to reflect the new context

## Memory Types

| Type | Description |
|------|-------------|
| **Atomic Notes** | Self-contained memory units with structured attributes |
| **Links** | Bidirectional connections between related memories |
| **Boxes** | Logical groupings of related notes (Zettelkasten concept) |
| **Context** | Evolving contextual descriptions that update over time |

## Storage Backends

| Component | Technology |
|-----------|------------|
| **Vector Store** | ChromaDB |
| **Embeddings** | Configurable text encoding model |
| **Metadata** | ChromaDB metadata fields (keywords, tags, links) |

ChromaDB handles both embedding storage and metadata persistence. No additional databases required.

## Retrieval Methods

1. **Semantic Embedding Search** -- Query ChromaDB for similar memory vectors
2. **Linked Memory Expansion** -- When a relevant memory is found, its linked memories within the same "box" are automatically retrieved
3. **Keyword/Tag Filtering** -- Metadata-based filtering for precise matches

The linked expansion is the key differentiator: finding one relevant memory pulls in its entire connected neighborhood, providing richer context than isolated vector retrieval.

## Agent Integration

- **Python SDK** -- Direct library usage
- **MCP Server** -- [a-mem-mcp-server](https://github.com/tobs-code/a-mem-mcp-server) (community) with dual-storage ChromaDB + NetworkX DiGraph
- **Multi-LLM** -- Supports different LLM backends for memory operations

```python
from a_mem import AMemory

mem = AMemory(llm_client=openai_client)

# Add memory (triggers note generation + connection analysis)
mem.add("Alice joined the ML team and will lead the recommendation project")

# Later addition triggers memory evolution
mem.add("The recommendation project deadline was moved to Q3")
# System links this to Alice's memory, updates context

# Retrieval with linked expansion
results = mem.search("What is Alice working on?")
# Returns Alice's note + linked project deadline note
```

## Benchmarks

The paper reports "superior improvement against existing SOTA baselines" across six foundation models, but specific numeric scores on standardized benchmarks (LoCoMo, LongMemEval) are not prominently published in the README.

The NeurIPS 2025 acceptance validates the approach's novelty and rigor within the academic community.

## Strengths

- Zettelkasten-inspired dynamic organization is a genuinely novel approach to agent memory
- Memory evolution: existing memories update when new related information arrives
- Linked expansion provides richer context than isolated vector retrieval
- NeurIPS 2025 acceptance validates academic rigor
- MIT license allows broad use
- Simple ChromaDB-only dependency (no complex multi-database setup)
- Community MCP server enables IDE integration
- LLM-agnostic design

## Weaknesses

- Small community (929 stars) compared to production-focused alternatives
- ChromaDB-only storage limits scalability options
- LLM-driven connection analysis adds latency and cost to every write
- Memory evolution (updating old memories) risks introducing errors
- No temporal reasoning built in
- No multimodal support
- Limited benchmark transparency (no public LoCoMo/LongMemEval numbers)
- Research-oriented; production hardening unclear
- MCP server is community-maintained, not official

## Usage Example

```python
from a_mem import AMemory
from openai import OpenAI

# Initialize with LLM backend
client = OpenAI()
mem = AMemory(
    llm_client=client,
    model="gpt-4o",
    persist_directory="./agent_memory"
)

# Add memories -- each triggers Zettelkasten organization
mem.add("User is a senior ML engineer at TechCorp")
mem.add("User is working on a transformer-based recommendation system")
mem.add("User prefers PyTorch over TensorFlow")
# System automatically links: ML engineer <-> recommendation system <-> PyTorch

# Add new info -- triggers memory evolution
mem.add("TechCorp is migrating to JAX for performance reasons")
# System updates context on PyTorch preference note (potential conflict)
# Links JAX migration to recommendation system and ML engineer notes

# Query with linked expansion
results = mem.search("What tools does the user work with?")
# Returns: PyTorch preference + JAX migration + recommendation system context
# All linked memories retrieved together for comprehensive context
```
