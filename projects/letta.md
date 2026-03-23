# Letta (formerly MemGPT)

- **GitHub**: https://github.com/letta-ai/letta
- **Website**: https://www.letta.com/
- **Documentation**: https://docs.letta.com/
- **License**: Apache 2.0 (true open source)
- **Paper**: [MemGPT: Towards LLMs as Operating Systems](https://arxiv.org/abs/2310.08560)
- **Related repos**:
  - [letta-code](https://github.com/letta-ai/letta-code) -- The memory-first coding agent
  - [claude-subconscious](https://github.com/letta-ai/claude-subconscious) -- Persistent memory layer for Claude Code
  - [skills](https://github.com/letta-ai/skills) -- Shared skill repository for Letta Code, Claude Code, Codex CLI
  - [letta-cowork](https://github.com/letta-ai/letta-cowork) -- Claude Cowork with stateful learning agent

## Summary

Letta is the most mature agent memory framework, originating from the MemGPT research paper that introduced the paradigm of LLMs as operating systems managing their own memory. Unlike systems that add memory as an external retrieval layer, Letta agents actively manage their own context window -- deciding what to remember, what to archive, and what to forget using tool calls. The agent treats in-context memory like RAM and archival storage like disk, with explicit read/write operations. In 2026, the ecosystem expanded significantly with Letta Code (a memory-first coding CLI), claude-subconscious (injecting Letta memory into Claude Code sessions), and a shared skills repository compatible with Claude Code and Codex CLI. The "Codex CLI + Letta Code daily driver" workflow has gained traction on X among power users.

## Architecture Overview

### LLM-as-Operating-System Paradigm

The core insight of MemGPT/Letta is that the LLM itself should manage memory, rather than having an external system decide what's relevant. The agent is given tools to read and write its own context, mimicking how an OS manages RAM, cache, and disk.

### Three-Tier Memory Hierarchy

```
+--------------------------------------------------+
|  CONTEXT WINDOW (what the LLM sees each turn)    |
|                                                   |
|  +--------------------------------------------+  |
|  | Core Memory (RAM)                           |  |
|  | - Persona block: agent identity/personality |  |
|  | - Human block: user details and preferences |  |
|  | - Custom blocks: project context, etc.      |  |
|  +--------------------------------------------+  |
|                                                   |
|  System prompt + current conversation turns       |
+--------------------------------------------------+
         |                          |
         v                          v
+------------------+    +----------------------+
| Recall Memory    |    | Archival Memory       |
| (Disk Cache)     |    | (Cold Storage)        |
| - Full convo     |    | - Large documents     |
|   history        |    | - Knowledge bases     |
| - Searchable     |    | - Insert & query      |
|   by the agent   |    |   via tool calls      |
+------------------+    +----------------------+
```

**Core Memory** -- Small structured blocks that live permanently in the agent's context window. The agent reads these every turn and can modify them with `core_memory_replace` and `core_memory_append` tool calls. Typically contains persona definition, user profile, and project-specific context.

**Recall Memory** -- Complete conversation history stored outside the context window. The agent can search it with `conversation_search` and `conversation_search_date` when it needs to look back at prior interactions. Acts like a disk cache.

**Archival Memory** -- Long-term storage for large volumes of information. The agent uses `archival_memory_insert` to store information and `archival_memory_search` to query it. Backed by vector search (pgvector in production).

### Self-Editing Mechanism

The critical design choice: **the agent decides what to remember**. When the agent encounters something important, it calls a memory tool to write it to the appropriate tier. This means:

- Memory is intentional, not automatic
- The agent can update/overwrite its own persona or user profile mid-conversation
- Context window usage is optimized by the agent itself
- The agent can explicitly "forget" by overwriting core memory blocks

### Agent Execution Loop

Each turn, the agent:
1. Reads system prompt + core memory blocks
2. Sees recent conversation turns
3. Decides whether to use memory tools (search recall, search/insert archival, update core)
4. Generates response
5. Memory state persists to next turn

## Memory Types Supported

| Type | Tier | Persistence | Access Method |
|------|------|-------------|---------------|
| **Persona** | Core (in-context) | Permanent, agent-editable | Always visible; `core_memory_replace` |
| **Human/User profile** | Core (in-context) | Permanent, agent-editable | Always visible; `core_memory_replace` |
| **Custom blocks** | Core (in-context) | Permanent, agent-editable | Always visible; `core_memory_append` |
| **Conversation history** | Recall (out-of-context) | Permanent | `conversation_search`, `conversation_search_date` |
| **Long-term knowledge** | Archival (out-of-context) | Permanent | `archival_memory_insert`, `archival_memory_search` |

## Storage Backends

| Backend | Use Case | Notes |
|---------|----------|-------|
| **SQLite** | Development, local use | Default for `pip install` |
| **PostgreSQL + pgvector** | Production | Default for Docker deployment; recommended |
| **Letta Cloud** | Managed hosting | Hosted by Letta Inc. |

The storage layer manages agent state, tool definitions, memory blocks, conversation history, and archival embeddings. PostgreSQL with pgvector is the recommended production backend.

## Retrieval Methods

| Method | Description |
|--------|-------------|
| **Core memory read** | Direct access -- always in context window |
| **Conversation search** | Keyword/semantic search over recall memory |
| **Date-based search** | Temporal filtering on conversation history |
| **Archival search** | Vector similarity search over archival memory (pgvector) |
| **Agent-driven** | The agent itself decides when and what to search |

Unlike Hindsight or Supermemory, retrieval in Letta is **agent-initiated** -- the LLM decides when to search memory rather than having an external system inject context. This gives the agent more autonomy but means retrieval quality depends on the agent's judgment.

## Agent/CLI Integration

### Letta Code

[letta-code](https://github.com/letta-ai/letta-code) is a memory-first coding agent CLI built on the Letta framework.

Key commands:
- **`/init`** -- Initialize memory for the current project
- **`/remember [instruction]`** -- Explicitly tell the agent to remember something; corrects agent behavior for future sessions
- **`/skill [instructions]`** -- Learn a reusable skill from the current conversation trajectory

The agent remembers your preferences, coding patterns, project structure, and past decisions across sessions without requiring external plugins.

### Claude Subconscious

[claude-subconscious](https://github.com/letta-ai/claude-subconscious) adds a persistent memory layer underneath Claude Code:

- A **Letta agent observes every Claude Code conversation** in the background
- Accumulates patterns, preferences, and knowledge across sessions, projects, and time
- Injects relevant context back into Claude Code's system prompt
- Claude Code "forgets everything between sessions" -- claude-subconscious fills that gap

This is an experimental project but represents a novel approach: rather than Claude Code managing its own memory, a separate stateful Letta agent acts as its "subconscious."

### Skills Repository

[letta-ai/skills](https://github.com/letta-ai/skills) is a shared repository of reusable skills compatible with:
- Letta Code
- Claude Code
- Codex CLI
- Any agent that supports `.skills` directories

Skills are modular capabilities the agent can learn and reuse. The skill-creator pattern (`/skill`) lets the agent distill its current work into a reusable skill automatically.

### Letta Cowork

[letta-cowork](https://github.com/letta-ai/letta-cowork) is Claude Cowork rebuilt with a stateful Letta agent that learns and grows over time, built on the Letta Code SDK.

## Performance Benchmarks

Letta does not publish LongMemEval scores in the same way as Hindsight or Supermemory. The system is architecturally different -- memory quality depends heavily on the underlying LLM's ability to make good memory management decisions.

From the [DEV Community comparison (2026)](https://dev.to/varun_pratapbhardwaj_b13/5-ai-agent-memory-systems-compared-mem0-zep-letta-supermemory-superlocalmemory-2026-benchmark-59p3):

| Aspect | Letta | Notes |
|--------|-------|-------|
| **Memory accuracy** | Depends on LLM | Agent-driven; no fixed retrieval pipeline score |
| **Context utilization** | High | Self-editing means efficient context window use |
| **Multi-session coherence** | Strong | Core memory persists and evolves across sessions |
| **Scalability** | Production-grade | PostgreSQL backend, Docker deployment |

The tradeoff is clear: Letta gives the agent maximum autonomy over memory at the cost of deterministic benchmark performance.

## Strengths

- **Most mature ecosystem** -- originating from the 2023 MemGPT paper, with 100+ contributors and extensive documentation
- **True self-editing memory** -- the agent manages its own context, not an external system. This is philosophically the most "agentic" approach
- **Apache 2.0 license** -- genuine open source with no usage restrictions
- **Production-ready storage** -- PostgreSQL + pgvector for real deployments
- **Rich CLI integration ecosystem** -- Letta Code, claude-subconscious, shared skills repo, letta-cowork
- **Skill learning** -- `/skill` command lets agents distill work into reusable modules
- **claude-subconscious** is a unique approach: a background Letta agent that gives Claude Code persistent memory without modifying Claude Code itself
- **Framework-agnostic** -- works with any LLM provider
- **Three-tier hierarchy** maps cleanly to how developers think about memory (RAM/cache/disk)

## Weaknesses

- **Agent-dependent quality** -- memory management is only as good as the LLM's judgment; a weaker model will make worse memory decisions
- **No hybrid retrieval** -- archival search is vector-only (pgvector); lacks BM25, graph traversal, or temporal fusion that Hindsight provides
- **Higher latency per turn** -- each turn may involve multiple tool calls (search recall, search archival, update core) before generating a response
- **No published LongMemEval score** -- makes direct comparison with Hindsight (91.4%) and Supermemory (~99%) difficult
- **Requires server infrastructure** -- PostgreSQL for production, Docker recommended; heavier than a simple SDK
- **Core memory blocks are small** -- limited by context window; the agent must actively manage what stays in core vs. archival
- **claude-subconscious is experimental** -- promising but not production-stable as of March 2026
- **No automatic learning** -- unlike Supermemory, the agent must explicitly decide to remember; if it doesn't call the memory tool, information is lost

## Key Code Example

### Python SDK -- Creating a Stateful Agent

```python
from letta import create_client

# Connect to Letta server
client = create_client(base_url="http://localhost:8283")

# Create an agent with custom core memory
agent = client.create_agent(
    name="coding-assistant",
    memory_blocks=[
        {"label": "persona", "value": "I am a senior Python developer who prefers clean, typed code."},
        {"label": "human", "value": "User: Rob. Prefers TypeScript. Uses WSL Ubuntu on Windows 11."},
        {"label": "project", "value": "Working on cc-tg, an agent memory survey project."}
    ],
    tools=["archival_memory_insert", "archival_memory_search",
           "conversation_search", "core_memory_replace", "core_memory_append"]
)

# Send a message -- the agent may self-edit memory during response
response = client.send_message(
    agent_id=agent.id,
    message="I just switched from PostgreSQL to SQLite for local dev. Remember that."
)

# The agent will likely call core_memory_replace to update the project block:
# "project" -> "Working on cc-tg. Using SQLite for local dev (switched from PostgreSQL)."
print(response.messages)

# Later -- search archival memory
results = client.get_archival_memory(agent_id=agent.id, query="database choice")
for r in results:
    print(r.text)
```

### Letta Code CLI Session

```
$ letta-code

> /init
# Scans project, builds initial core memory blocks

> /remember Always run tests before committing
# Agent stores this as a persistent preference

> Fix the broken API endpoint for user deletion
# Agent works on fix, may insert findings into archival memory
# Next session, it remembers the fix and your testing preference

> /skill Extract the pattern I used for error handling into a reusable skill
# Agent creates a .skills/error-handling.md file
```

---

*Last updated: March 2026*
