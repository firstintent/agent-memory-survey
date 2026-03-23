# MemMachine

## Project Info

| Field | Value |
|-------|-------|
| **GitHub** | [MemMachine/MemMachine](https://github.com/MemMachine/MemMachine) |
| **Stars** | 5.3k |
| **License** | Apache 2.0 |
| **Language** | Python (95.6%), TypeScript (2.2%) |
| **Status** | Active (v0.2, 23 releases, 827 commits) |
| **Website** | [memmachine.ai](https://memmachine.ai/) |
| **Backed by** | MemVerge |

## Summary

MemMachine is an open-source long-term memory layer for AI agents, backed by MemVerge. It provides a cross-platform, LLM-agnostic memory system with two distinct memory types: Episodic Memory (graph-based conversational context) and Profile Memory (SQL-based long-term user facts). The system works seamlessly across major LLM providers (OpenAI, Claude, Gemini, Grok, Llama, DeepSeek, Qwen) and can be deployed on any cloud or on-premises. MemMachine v0.2 achieved industry-leading LoCoMo benchmark scores with approximately 80% reduction in token usage vs. Mem0.

## Architecture

```
Agent / Application
    |
    v
  MemMachine API (REST / Python SDK / MCP Server)
    |
    v
  Memory Processing Layer
    |  -- Extract episodic context from conversations
    |  -- Extract profile facts from interactions
    |  -- Working memory for current session
    |
    +--> Episodic Memory --> Neo4j (graph database)
    |    (conversational context, cross-session)
    |
    +--> Profile Memory --> SQL Database
    |    (user facts, preferences, long-term)
    |
    +--> Working Memory (in-session, ephemeral)
```

### Memory Processing

When interactions flow through MemMachine:
1. The system classifies information as either episodic (contextual, conversational) or profile (factual, persistent)
2. Episodic data is stored as graph nodes/edges in Neo4j, preserving conversational structure and cross-session relationships
3. Profile data is stored as structured records in SQL, enabling precise fact lookup
4. Working memory maintains the current session state and is discarded after the session ends

## Memory Types

| Type | Description | Storage | Persistence |
|------|-------------|---------|-------------|
| **Episodic** | Conversational context, dialogue history, cross-session threads | Neo4j (graph) | Persistent across sessions |
| **Profile** | User facts, preferences, biographical data | SQL database | Long-term persistent |
| **Working** | Current session state, active context | In-memory | Session-scoped (ephemeral) |
| **Agent** | Agent state that survives restarts, session changes, and model switches | Combined | Persistent |

## Storage Backends

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Graph DB** | Neo4j | Episodic memory (conversational graphs) |
| **SQL DB** | SQL (specific engine not constrained) | Profile memory (user facts) |
| **In-Memory** | Application state | Working memory |

## Retrieval Methods

1. **Graph Traversal** -- Navigate conversational context graphs in Neo4j for episodic retrieval
2. **SQL Queries** -- Structured lookups for profile facts and preferences
3. **Cross-Memory Synthesis** -- Combine episodic context with profile facts for comprehensive answers

## Agent Integration

- **REST API** -- Language-agnostic HTTP endpoints
- **Python SDK** -- Native Python library
- **MCP Server** -- Model Context Protocol for IDE integration
- **LLM Agnostic** -- OpenAI, Anthropic, Bedrock, Ollama, and any LLM provider

```python
from memmachine import MemMachine

mm = MemMachine(llm_provider="openai")

# Store episodic memory
mm.add_episode(
    "User discussed their frustration with the current onboarding flow",
    user_id="user_123",
    session_id="session_abc"
)

# Store profile memory
mm.add_profile("user_123", {"role": "PM", "company": "Acme", "preference": "async"})

# Query across both memory types
results = mm.query("What does this user care about?", user_id="user_123")
# Combines: episodic (onboarding frustration) + profile (PM role, async preference)
```

## Benchmarks

### LoCoMo Performance (v0.2, December 2025)

MemMachine v0.2 achieved industry-leading LoCoMo benchmark scores:
- Best-in-class scores using GPT-4.1-mini and GPT-4o-mini
- ~80% reduction in token usage compared to Mem0
- Detailed numeric scores published on [memmachine.ai blog](https://memmachine.ai/blog/2025/12/memmachine-v0.2-delivers-top-scores-and-efficiency-on-locomo-benchmark/)

## Strengths

- Clean separation of episodic (graph) vs. profile (SQL) memory is architecturally sound
- Broadest LLM compatibility (8+ providers including open-source models)
- 80% token reduction vs. Mem0 demonstrates efficiency
- Enterprise backing by MemVerge provides sustainability
- Three access modes: REST API, Python SDK, MCP Server
- Working memory concept handles session state cleanly
- Agent memory persists across restarts and model changes
- Active release cadence (23 releases, 827 commits)

## Weaknesses

- Neo4j dependency adds operational complexity
- Moderate community size (5.3k stars) for an enterprise-backed project
- No vector database component; relies on graph + SQL only
- No built-in temporal validity tracking (unlike Zep)
- No multimodal support
- No published standardized benchmark numbers in README (blog-only)
- SQL backend for profiles may not scale for very high-cardinality entity sets
- Community-driven but corporate-backed; long-term open-source commitment unclear

## Usage Example

```python
from memmachine import MemMachine

# Initialize with multi-LLM support
mm = MemMachine(
    graph_db_url="bolt://localhost:7687",
    sql_db_url="sqlite:///profiles.db",
    llm_provider="anthropic",
    model="claude-sonnet-4-20250514"
)

# Episodic memory: store conversational context
mm.add_episode(
    content="Alice mentioned the API redesign is behind schedule by 2 weeks",
    user_id="alice",
    session_id="standup_march_20"
)

mm.add_episode(
    content="Alice proposed splitting the API work into microservices",
    user_id="alice",
    session_id="architecture_review"
)

# Profile memory: store persistent facts
mm.add_profile("alice", {
    "role": "Senior Backend Engineer",
    "team": "Platform",
    "expertise": ["API design", "distributed systems"],
    "communication_style": "direct, prefers written over verbal"
})

# Cross-memory query
results = mm.query("Summarize Alice's current work situation", user_id="alice")
# Episodic: API redesign behind schedule, proposed microservice split
# Profile: Senior Backend on Platform team, API design expert
# Synthesized: "Alice, a senior backend engineer specializing in API design,
#   is navigating an API redesign that's behind schedule and has proposed
#   a microservices architecture as a solution."

# Switch LLM provider seamlessly
mm.set_provider("ollama", model="llama3.1")
results = mm.query("What is Alice's expertise?", user_id="alice")
```
