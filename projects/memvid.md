# Memvid

## Project Info

| Field | Value |
|-------|-------|
| **GitHub** | [memvid/memvid](https://github.com/memvid/memvid) |
| **Stars** | 13.6k |
| **License** | Apache 2.0 |
| **Language** | Rust (core), Python + Node.js SDKs |
| **Status** | Active (v2.0 on crates.io), Rust-native |
| **Website** | [memvid.com](https://memvid.com) |
| **Docs** | [docs.memvid.com](https://docs.memvid.com) |

## Summary

Memvid is a serverless, single-file memory layer for AI agents. It replaces complex RAG pipelines and database dependencies with a single `.mv2` file that stores, indexes, and retrieves knowledge. Inspired by video encoding concepts, Memvid organizes memory as append-only sequences of "Smart Frames" -- immutable units containing content, timestamps, checksums, and metadata. The system was rewritten from Python to Rust, delivering sub-millisecond retrieval (0.025ms P50) and best-in-class LoCoMo benchmark results (+35% SOTA).

## Architecture

```
Input Data (text, audio, images, video)
    |
    v
  Smart Frame Encoding
    |  -- Content + timestamp + checksum + metadata
    |  -- Immutable, append-only
    |
    v
  .mv2 File (single portable capsule)
    |  -- Data segments
    |  -- Embedding index (HNSW)
    |  -- BM25 lexical index (Tantivy)
    |  -- Metadata catalog
    |  -- Temporal track
    |
    v
  Retrieval Engine
    +--> HNSW vector similarity (vec feature)
    +--> BM25 full-text search (lex feature)
    +--> CLIP visual search (clip feature)
    +--> Whisper audio search (whisper feature)
    +--> Temporal range queries (temporal_track feature)
```

### Smart Frame Design

A Smart Frame is the atomic unit of Memvid memory:
- **Immutable** -- Once written, never modified (append-only log)
- **Self-contained** -- Content + embeddings + metadata in one unit
- **Grouped** -- Frames are organized into segments for efficient compression, indexing, and parallel reads
- **Versioned** -- Supports queries over past memory states and timeline inspection

The `.mv2e` variant adds password-based encryption for sensitive data.

## Memory Types

| Type | Description | Feature Flag |
|------|-------------|-------------|
| **Textual** | Natural language content with embeddings | Default |
| **Visual** | Image content with CLIP embeddings | `clip` |
| **Audio** | Transcribed audio with Whisper | `whisper` |
| **Temporal** | Time-indexed frames for historical queries | `temporal_track` |
| **Encrypted** | Password-protected memory capsules | `encryption` |

## Storage Backends

There is exactly one storage backend: the `.mv2` file itself. This is the core design philosophy -- no external databases, no server processes, no configuration. The file is:
- Portable (git commit, scp, share)
- Self-contained (data + indexes + metadata)
- Append-only (safe concurrent reads)
- Optionally encrypted (`.mv2e`)

## Retrieval Methods

| Method | Technology | Feature Flag | Latency |
|--------|-----------|-------------|---------|
| **Semantic** | HNSW + ONNX embeddings | `vec` | 0.025ms P50 |
| **Full-text** | Tantivy BM25 | `lex` | Sub-ms |
| **Visual** | CLIP similarity | `clip` | -- |
| **Audio** | Whisper transcription + search | `whisper` | -- |
| **Temporal** | Range queries over time axis | `temporal_track` | -- |
| **Hybrid** | Fused scoring across channels | Combined | 0.075ms P99 |

1,372x higher throughput than standard RAG approaches.

## Agent Integration

- **Rust crate** -- `memvid-core` v2.0 on crates.io
- **Python SDK** -- `pip install memvid`
- **Node.js SDK** -- npm package
- **CLI** -- `memvid-cli` for command-line operations
- **Claude Brain** -- [memvid/claude-brain](https://github.com/memvid/claude-brain) -- give Claude Code photographic memory in one `.mv2` file

```python
import memvid

# Create a memory capsule
mem = memvid.MemvidWriter("project_knowledge.mv2")
mem.add("Alice is the tech lead for Project X")
mem.add("Sprint deadline is March 15, 2025")
mem.build()

# Query the capsule
reader = memvid.MemvidReader("project_knowledge.mv2")
results = reader.search("Who leads Project X?")
```

## Benchmarks

| Metric | Score | Comparison |
|--------|-------|------------|
| **LoCoMo Overall** | +35% SOTA | Best-in-class long-horizon conversational recall |
| **Multi-hop Reasoning** | +76% | vs. industry average |
| **Temporal Reasoning** | +56% | vs. industry average |
| **P50 Latency** | 0.025ms | Sub-millisecond |
| **P99 Latency** | 0.075ms | Sub-millisecond |
| **Throughput** | 1,372x | vs. standard approaches |

Evaluated on LoCoMo (10 x ~26K-token conversations) with open-source eval and LLM-as-Judge methodology.

## Strengths

- Zero infrastructure: single file, no databases, no servers
- Exceptional performance: 0.025ms P50, 1,372x throughput improvement
- Best-in-class LoCoMo scores (+35% SOTA, +76% multi-hop)
- Rust core with Python/Node.js/CLI access -- production-grade performance with developer-friendly APIs
- Multimodal: text, images (CLIP), audio (Whisper)
- Portable: git-committable, shareable `.mv2` files
- Append-only design enables temporal queries and historical inspection
- Encryption support for sensitive data
- Claude Code integration via claude-brain

## Weaknesses

- Single-file design has scalability ceiling for very large datasets
- No graph database component for complex relational queries
- Append-only means no in-place updates; superseded info stays in the file
- Relatively new Rust rewrite; ecosystem maturity still developing
- No built-in entity extraction or knowledge graph construction
- No multi-agent memory sharing (each agent gets its own .mv2)
- Feature flags add complexity (must opt in to multimodal, encryption, etc.)
- No SaaS offering; self-hosted only

## Usage Example

```rust
// Rust API
use memvid_core::{MemvidWriter, MemvidReader, SearchOptions};

// Write memories
let mut writer = MemvidWriter::new("agent_memory.mv2")?;
writer.add_text("User prefers Python and works at Acme Corp")?;
writer.add_text("Meeting scheduled for 2025-03-15 at 10am")?;
writer.build()?;

// Read and search
let reader = MemvidReader::open("agent_memory.mv2")?;
let results = reader.search(
    "When is the next meeting?",
    SearchOptions::default()
)?;
// Returns in 0.025ms: "Meeting scheduled for 2025-03-15 at 10am"
```

```bash
# CLI usage
memvid-cli create agent_memory.mv2 --input ./documents/
memvid-cli search agent_memory.mv2 "What does the user prefer?"
memvid-cli info agent_memory.mv2  # Show stats and frame count
```
