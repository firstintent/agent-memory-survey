# Google Always-On Memory Agent

## Project Info

| Field | Value |
|-------|-------|
| **GitHub** | [GoogleCloudPlatform/generative-ai/.../always-on-memory-agent](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/gemini/agents/always-on-memory-agent) |
| **License** | MIT |
| **Language** | Python |
| **Status** | Active, reference implementation |
| **Framework** | Google Agent Development Kit (ADK) |
| **Model** | Gemini 3.1 Flash-Lite |

## Summary

Google's Always-On Memory Agent is a vector-DB-free, LLM-driven memory system that uses plain SQLite for storage and Gemini's multimodal capabilities for extraction. It intentionally ditches vector databases and embeddings in favor of a provocative design: "No vector database. No embeddings. Just an LLM that reads, thinks, and writes structured memory." The system runs continuously, ingests 27 file types across 5 categories (text, image, audio, video, documents), and performs automatic memory consolidation on a configurable timer. Released under MIT license, it serves as a practical reference for teams wanting persistent agent memory without RAG infrastructure.

## Architecture

```
Input Sources (27 file types)
    |
    v
  Multi-Agent Orchestration (Google ADK)
    |
    +--> IngestAgent
    |    -- Process multimodal inputs
    |    -- Extract entities, topics, importance
    |    -- Write structured memories to SQLite
    |
    +--> ConsolidateAgent (every 30 min, configurable)
    |    -- Review unconsolidated memories
    |    -- Find cross-memory connections
    |    -- Merge duplicates, drop noise
    |    -- Generate consolidated insights
    |
    +--> QueryAgent
    |    -- Read relevant memories from SQLite
    |    -- Synthesize answers with citations
    |    -- Return sourced responses
    |
    v
  SQLite (memory.db)
    |
  Streamlit Dashboard + HTTP API
```

### Design Philosophy

The system works like human sleep-based memory consolidation:
1. **Waking phase** -- IngestAgent continuously processes new inputs, creating raw memories
2. **Sleep phase** -- ConsolidateAgent periodically reviews, connects, deduplicates, and compresses memories
3. **Recall phase** -- QueryAgent reads the consolidated store and synthesizes answers

The LLM itself acts as the retrieval engine. Instead of vector similarity search, the QueryAgent reads relevant memories from SQLite and uses Gemini's reasoning to find connections and synthesize responses.

## Memory Types

| Type | Description |
|------|-------------|
| **Raw Memories** | Structured extracts from ingested content (entities, topics, importance scores) |
| **Consolidated Insights** | Cross-memory connections and synthesized knowledge |
| **Source Records** | Processed file metadata and provenance |

## Storage Backends

| Component | Technology |
|-----------|------------|
| **Primary** | SQLite (`memory.db`) |
| **No Vector DB** | By design |
| **No Embeddings** | By design |

The entire memory store is a single SQLite file. This eliminates vector database setup, embedding model configuration, index tuning, and similarity search calibration.

## Retrieval Methods

1. **LLM-Driven Retrieval** -- The QueryAgent reads memories from SQLite and uses Gemini's reasoning to identify relevant information. No embedding similarity; the LLM itself determines relevance.
2. **Consolidation-Enhanced** -- Periodic consolidation creates cross-memory insights that improve retrieval quality over time.
3. **Citation-Backed** -- Responses include citations back to source memories.

This is fundamentally different from every other system in this survey. There is no vector search, no BM25, no graph traversal. The LLM reads the memory store directly.

## Multimodal Support

27 file types across 5 categories:

| Category | Formats |
|----------|---------|
| **Text** | `.txt`, `.md`, `.json`, `.csv`, `.log`, `.xml`, `.yaml`, `.yml` |
| **Images** | `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, `.bmp`, `.svg` |
| **Audio** | `.mp3`, `.wav`, `.ogg`, `.flac`, `.m4a`, `.aac` |
| **Video** | `.mp4`, `.webm`, `.mov`, `.avi`, `.mkv` |
| **Documents** | `.pdf` |

Gemini's native multimodal API extracts structured summaries, entities, topics, and importance scores from all formats.

## Agent Integration

- **HTTP API** -- Local REST endpoints
- **Streamlit Dashboard** -- Visual memory management
- **Google ADK** -- Built on Google's Agent Development Kit

### API Endpoints

| Endpoint | Method | Function |
|----------|--------|----------|
| `/status` | GET | Memory statistics |
| `/memories` | GET | List stored memories |
| `/ingest` | POST | Add text content |
| `/query?q=...` | GET | Query with natural language |
| `/consolidate` | POST | Trigger manual consolidation |
| `/delete` | POST | Remove specific memory |
| `/clear` | POST | Full memory reset |

## Benchmarks

No published LoCoMo or LongMemEval benchmark scores. The project is positioned as a reference implementation demonstrating an alternative architectural philosophy rather than a benchmark competitor.

The key claim is operational simplicity: comparable memory quality with dramatically reduced infrastructure complexity.

## Strengths

- Radically simple: SQLite only, no vector DB, no embeddings, no graph DB
- Broadest multimodal support (27 file types, 5 categories) of any system surveyed
- MIT license with explicit commercial use allowance
- Google backing and ADK integration
- Consolidation mechanism mimics human memory processing elegantly
- Zero infrastructure setup beyond Python + SQLite
- Streamlit dashboard for visual management
- Clean REST API for integration
- Low cost: Flash-Lite is Google's cheapest Gemini model

## Weaknesses

- LLM-driven retrieval does not scale: reading all memories for every query has O(n) cost
- No vector search means no sub-linear retrieval; performance degrades with memory size
- Gemini dependency (Flash-Lite) limits portability to other LLM providers
- No published benchmarks make quality comparison impossible
- Consolidation timer (30 min default) means recent memories may not be connected yet
- SQLite single-file bottleneck for concurrent writes
- No MCP server or agent framework integrations (LangChain, CrewAI, etc.)
- Reference implementation, not a production-hardened library
- Google ADK is relatively new; ecosystem still developing
- No multi-agent memory sharing

## Usage Example

```python
# Start the agent
# python -m always_on_memory_agent

# Ingest via API
import requests

# Add text
requests.post("http://localhost:8000/ingest", json={
    "content": "Alice is the PM for Project Atlas. Launch date is Q2 2026."
})

# Add a file (multimodal)
with open("architecture_diagram.png", "rb") as f:
    requests.post("http://localhost:8000/ingest", files={"file": f})

# Add audio notes
with open("meeting_recording.mp3", "rb") as f:
    requests.post("http://localhost:8000/ingest", files={"file": f})

# Query (LLM reads memories directly, no vector search)
response = requests.get("http://localhost:8000/query", params={
    "q": "What is Project Atlas and when does it launch?"
})
print(response.json())
# {
#   "answer": "Project Atlas is led by Alice as PM, with a Q2 2026 launch date.",
#   "citations": ["memory_001", "memory_003"],
#   "confidence": 0.92
# }

# Trigger manual consolidation
requests.post("http://localhost:8000/consolidate")

# Check status
requests.get("http://localhost:8000/status")
# {"total_memories": 47, "consolidated": 35, "pending": 12}
```

```bash
# Or use the Streamlit dashboard
streamlit run dashboard.py
# Opens visual memory browser with ingest/query/consolidate controls
```
