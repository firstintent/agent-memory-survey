# Supermemory

- **GitHub**: https://github.com/supermemoryai/supermemory
- **Website**: https://supermemory.ai/
- **Documentation**: https://supermemory.ai/docs
- **License**: Open source (core engine); Pro plan required for hosted API and plugins
- **Research**: https://supermemory.ai/research/

## Summary

Supermemory is a memory engine and API that claims the top position on three major agent memory benchmarks: LongMemEval (~99%), LoCoMo, and ConvoMem. It takes a "post-vector" approach where LLM-powered compression and reasoning replace naive embedding similarity as the primary memory mechanism. Rather than just appending data, Supermemory builds a vector graph with ontology-aware edges that enables knowledge updates, merges, contradiction resolution, and inference. It ships with built-in plugins for Claude Code and OpenCode, supports ingestion of PDFs, web pages, images, audio, and chat logs, and automatically learns from conversations to build deep user profiles. The system handles superseding information gracefully -- if a user says "I moved to SF" after previously saying "I live in NYC", the old fact is updated rather than creating a contradictory duplicate.

## Architecture Overview

### Post-Vector Engine

Supermemory's core innovation is moving beyond pure vector similarity. The internal architecture consists of:

1. **Vector Graph Engine** -- A custom graph structure with ontology-aware edges. Unlike a flat vector store, nodes are connected by typed relationships (contradicts, supersedes, supports, elaborates). This allows the system to resolve conflicts and track knowledge evolution.

2. **LLM Compression Layer** -- Instead of storing raw chunks, incoming content is processed through an LLM that extracts facts, entities, relationships, and temporal markers. This compressed representation is what gets stored and indexed.

3. **Ontology-Aware Edges** -- Relationships between memory nodes carry semantic types, enabling operations like:
   - **Supersession**: New facts replace outdated ones
   - **Merging**: Related facts from different sessions are combined
   - **Contradiction detection**: Conflicting information is flagged and resolved
   - **Inference**: New knowledge is derived from existing connections

4. **User Profile Builder** -- An internal model builds deep user profiles from behavior patterns, preferences, and stated facts. The AI doesn't just recall -- it understands intent, preferences, and context.

### Retrieval Pipeline

- Hybrid vector + keyword search with **sub-300ms latency**
- Context-aware reranking to surface the most relevant memories
- Smart chunking that preserves meaning across document boundaries
- Temporal reasoning for time-dependent queries

### Content Processing

Supermemory can understand and ingest:
- PDFs and documents
- Web pages
- Markdown files
- Images and audio
- Email threads
- Chat logs and conversation transcripts
- Video transcripts

## Memory Types Supported

| Type | Description |
|------|-------------|
| **Episodic** | Conversation history and interaction events |
| **Semantic** | Extracted facts, entities, and relationships |
| **User Profile** | Preferences, behavior patterns, stated context |
| **Project Knowledge** | Codebase structure, conventions, architecture decisions |
| **Temporal** | Time-stamped facts with supersession tracking |

Supermemory does not expose these as separate stores to the user -- they are unified internally and the retrieval system selects from the appropriate type based on query context.

## Storage Backends

- **Supermemory Cloud** (primary) -- Managed hosted service via supermemory.ai
- **Self-hosted** -- The core engine is open source on GitHub
- Underlying storage uses a custom vector graph engine (not a standard vector database)

## Retrieval Methods

| Method | Description |
|--------|-------------|
| **Hybrid search** | Vector similarity + keyword matching |
| **Graph traversal** | Walk ontology-aware edges between memory nodes |
| **Temporal reasoning** | Time-aware queries with supersession resolution |
| **Context-aware reranking** | LLM-powered relevance scoring |
| **Profile-aware retrieval** | Results weighted by user profile and preferences |

## Agent/CLI Integration

### Claude Code Plugin

Official first-party plugin with one-command installation:

```bash
# Add the plugin marketplace
/plugin marketplace add supermemoryai/claude-supermemory

# Install the plugin
/plugin install claude-supermemory

# Initialize -- scans your project and builds initial memory
/supermemory-init
```

**Features:**
- **Context injection**: On session start, relevant memories are automatically fetched and injected into Claude's context
- **Auto-capture**: Tool usage and conversation patterns are captured during sessions
- **Privacy protection**: Wrap content in `<private>` tags and it's redacted before storage (API keys, credentials, personal notes)
- **Configurable**: Similarity threshold, max memories per request, keyword patterns for memory detection

**Repository**: https://github.com/supermemoryai/claude-supermemory

### OpenCode Plugin

Dedicated plugin for OpenCode:

```
# Install via OpenCode plugin system
```

**Repository**: https://github.com/supermemoryai/opencode-supermemory

### OpenClaw Plugin

Also ships a plugin for OpenClaw agents.

### API Integration

Any agent can integrate via the REST API:

```bash
curl -X POST https://api.supermemory.ai/v1/memories \
  -H "Authorization: Bearer $SUPERMEMORY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"content": "User prefers TypeScript over JavaScript"}'
```

## Performance Benchmarks

| Benchmark | Supermemory | Hindsight | Zep | Mem0 |
|-----------|-------------|-----------|-----|------|
| **LongMemEval** | **~99%** | 91.4% | 71% | 49% |
| **LoCoMo** | **#1** | -- | -- | -- |
| **ConvoMem** | **#1** | -- | -- | -- |

Supermemory claims #1 on all three major benchmarks. The LongMemEval result of ~99% represents a significant jump from Hindsight's 91.4%, which was itself the first to break 90%.

Key benchmark categories where Supermemory excels:
- **Multi-session reasoning**: 71.43% (historically difficult for vector-only approaches)
- **Temporal reasoning**: 76.69% (requires understanding time and supersession)
- **High-noise environments**: Handles 115k+ token contexts effectively

## Strengths

- **Benchmark dominance** -- #1 on LongMemEval, LoCoMo, and ConvoMem simultaneously
- **Post-vector approach** solves fundamental problems with pure embedding similarity (contradictions, supersession, temporal reasoning)
- **Built-in Claude Code plugin** with one-command install -- lowest friction path to agent memory
- **Automatic learning** from conversations without explicit "remember this" commands
- **Supersession handling** ("moved to SF" correctly replaces "live in NYC") -- critical for long-lived agents
- **Multi-format ingestion** (PDFs, web pages, images, audio, video transcripts, emails)
- **Privacy controls** with `<private>` tag redaction
- **Sub-300ms retrieval latency**
- **Deep user profiling** builds understanding of intent and preferences, not just facts

## Weaknesses

- **Cloud dependency** -- while the core engine is open source, the full feature set (plugins, managed API) requires the Pro plan and cloud connectivity
- **Less transparent architecture** -- the "post-vector" engine is less well-documented academically than Hindsight's TEMPR/CARA (no published paper with full details)
- **Vendor lock-in risk** -- heavy reliance on Supermemory's API and cloud infrastructure
- **Cost** -- Pro plan required for plugin access and full API; not fully free for serious use
- **Newer project** with less production track record than Letta/MemGPT
- **Benchmark methodology** -- self-reported results without independent reproduction (as of March 2026)
- **No explicit belief tracking** -- unlike Hindsight's evolving beliefs with confidence scores, supersession is implicit

## Key Code Example

### Claude Code Integration (Primary Use Case)

After installing the plugin:

```
# In Claude Code session:

> /supermemory-init
# Agent explores project: reads package.json, scans directory structure,
# checks commit history, learns conventions

> Fix the auth bug in the login flow
# Claude works on the fix. Supermemory auto-captures:
# - Files modified and patterns used
# - Decisions made and reasons
# - User feedback and corrections

# Next session -- memories are auto-injected:
# "This project uses NextAuth with JWT strategy. User prefers
#  explicit error handling over try/catch wrappers. Auth routes
#  are in src/app/api/auth/. Last session fixed a token refresh
#  bug in the login flow."
```

### REST API Usage

```python
import requests

SUPERMEMORY_API_KEY = "sm_..."
BASE_URL = "https://api.supermemory.ai/v1"

# Add a memory
requests.post(f"{BASE_URL}/memories", headers={
    "Authorization": f"Bearer {SUPERMEMORY_API_KEY}"
}, json={
    "content": "The project uses PostgreSQL 16 with pgvector for embeddings",
    "metadata": {"project": "my-app", "category": "infrastructure"}
})

# Search memories
response = requests.post(f"{BASE_URL}/search", headers={
    "Authorization": f"Bearer {SUPERMEMORY_API_KEY}"
}, json={
    "query": "What database does the project use?",
    "limit": 5
})

for memory in response.json()["results"]:
    print(f"[{memory['score']:.2f}] {memory['content']}")
```

---

*Last updated: March 2026*
