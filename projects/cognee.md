# Cognee

## Project Info

| Field | Value |
|-------|-------|
| **GitHub** | [topoteretes/cognee](https://github.com/topoteretes/cognee) |
| **Stars** | 14.5k |
| **License** | Apache 2.0 |
| **Language** | Python |
| **Status** | Active, open-source with commercial offerings |
| **Website** | [cognee.ai](https://www.cognee.ai/) |

## Summary

Cognee is a knowledge engine for AI agent memory that replaces traditional RAG with scalable ECL (Extract, Cognify, Load) pipelines. It combines vector search, graph databases, and cognitive science approaches to make documents both searchable by meaning and connected by relationships. The system ingests data from 30+ sources, extracts knowledge graph triplets, and provides a unified memory architecture for agents that need to remember and reason across sessions.

## Architecture

```
Data Sources (30+ formats)
    |
    v
  EXTRACT -- Parse and chunk input data
    |
    v
  COGNIFY -- Build knowledge graph triplets
    |        -- Generate vector embeddings
    |        -- Apply ontology grounding
    |
    v
  LOAD -- Store in graph DB + vector DB
    |
    v
  Search API (graph traversal + semantic search)
```

The ECL pipeline is the core abstraction:

1. **Extract** -- Ingests data from files, APIs, databases, and other sources. Handles parsing, chunking, and normalization across 30+ data formats.
2. **Cognify** -- The intelligence layer. Extracts knowledge graph triplets (subject-predicate-object), generates embeddings, resolves entities, and grounds concepts against ontologies. This is where raw data becomes structured memory.
3. **Load** -- Persists processed data into both graph and vector storage backends, maintaining bidirectional links between the two representations.

The Memify Pipeline (introduced 2025) adds post-processing optimization: making memory smarter through iterative refinement, deduplication, and quality scoring.

## Memory Types

| Type | Description |
|------|-------------|
| **Knowledge Graph** | Entity-relationship triplets extracted from source data |
| **Semantic Embeddings** | Dense vector representations for similarity search |
| **Ontology-Grounded** | Concepts mapped to formal ontology structures |
| **Cross-Agent** | Shared memory enabling knowledge transfer between agents |

## Storage Backends

**Graph Databases:**
- Neo4j (primary)
- NetworkX (in-memory, development)

**Vector Databases:**
- Qdrant
- Weaviate
- pgvector
- LanceDB

**Relational:**
- PostgreSQL (metadata, provenance)
- SQLite (local development)

## Retrieval Methods

1. **Graph Traversal** -- Follow entity relationships through the knowledge graph
2. **Semantic Vector Search** -- Cosine similarity over embeddings
3. **Hybrid** -- Combined graph + vector retrieval
4. **Ontology-Guided** -- Use formal ontology structure to constrain and improve search

## Agent Integration

- **Python SDK** -- `cognee.add()`, `cognee.cognify()`, `cognee.search()`
- **CLI** -- `cognee-cli add`, `cognee-cli cognify`, `cognee-cli search`
- **REST API** -- HTTP endpoints for language-agnostic integration
- **LangChain** -- Compatible as a retrieval backend
- **Multi-Agent** -- Cross-agent knowledge sharing via shared memory spaces

```python
import cognee

# Ingest documents
await cognee.add("meeting_notes.pdf")
await cognee.add("customer_data.csv")

# Process into knowledge graph
await cognee.cognify()

# Query
results = await cognee.search("What decisions were made about customer onboarding?")
```

## Benchmarks

Cognee does not prominently publish standardized benchmark scores (e.g., LoCoMo, LongMemEval). The project focuses more on production readiness, data source breadth, and knowledge graph quality than competitive leaderboard positioning.

The 2025 research paper on knowledge graph optimization for LLM reasoning provides internal evaluation metrics, but these are not directly comparable to the LoCoMo/LongMemEval benchmarks used by competitors.

## Strengths

- 30+ data source connectors (broadest ingestion in the space)
- Clean ECL pipeline abstraction is easy to understand and extend
- Ontology grounding adds formal structure that pure extraction misses
- Multiple graph and vector backend options
- Cross-agent knowledge sharing is a unique capability
- Active development with Memify post-processing pipeline
- GitHub Secure Open Source certified
- Good documentation and 6-line quickstart

## Weaknesses

- No published standardized benchmark scores makes comparison difficult
- Knowledge graph triplet extraction quality varies with LLM capability
- Complex pipeline means more moving parts to debug
- Graph + vector dual storage adds operational overhead
- Ontology grounding requires domain-specific configuration for best results
- Smaller community than Mem0 (14.5k vs 50.7k stars)
- No built-in temporal reasoning (unlike Zep)

## Usage Example

```python
import cognee

# Configure backends
cognee.config.set_vector_db("qdrant", url="http://localhost:6333")
cognee.config.set_graph_db("neo4j", url="bolt://localhost:7687")

# Ingest from multiple sources
await cognee.add("quarterly_report.pdf")
await cognee.add("slack_export.json")
await cognee.add("https://api.example.com/customers", source_type="api")

# Run the ECL pipeline
await cognee.cognify()

# Search with graph-aware retrieval
results = await cognee.search(
    "Which customers mentioned in Slack were also in the quarterly report?",
    search_type="graph"
)

for result in results:
    print(f"{result.content} (confidence: {result.score:.2f})")
    print(f"  Source: {result.provenance}")
    print(f"  Related entities: {result.entities}")
```
