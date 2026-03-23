# MemOS

## Project Info

| Field | Value |
|-------|-------|
| **GitHub** | [MemTensor/MemOS](https://github.com/MemTensor/MemOS) |
| **Stars** | 7.6k |
| **License** | Apache 2.0 |
| **Language** | Python |
| **Status** | Active (v2.0 "Stardust" released Dec 2025) |
| **Paper** | [arXiv:2505.22101](https://arxiv.org/abs/2505.22101) (May 2025), [arXiv:2507.03724](https://arxiv.org/abs/2507.03724) (July 2025) |
| **PyPI** | `pip install MemoryOS` |

## Summary

MemOS is an AI memory operating system for LLM and agent systems that introduces the concept of persistent skill memory for cross-task skill reuse and evolution. Rather than treating memory as just fact storage, MemOS classifies all AI-related knowledge into three interoperable forms: Plaintext Memory (base layer), Activation Memory (hot/fast access), and Parametric Memory (internalized into model weights). The system's signature innovation is MemCube -- a composable memory structure enabling isolation, sharing, and dynamic composition of knowledge across users, tasks, and agents.

## Architecture

```
Users / Agents / Tasks
    |
    v
  Unified Memory API
    |
    v
  MemCube (composable memory structure)
    |  -- Isolation per user/agent
    |  -- Controlled sharing across boundaries
    |  -- Dynamic composition of knowledge bases
    |
    v
  Three Memory Forms:
    +--> Plaintext Memory (text, documents -- base layer)
    +--> Activation Memory (hot embeddings -- fast access, like RAM)
    +--> Parametric Memory (model weights -- internalized, like CPU cache)
    |
    v
  Storage Layer:
    +--> Qdrant (vector embeddings)
    +--> Neo4j (knowledge graph)
    +--> SQLite (local metadata)
    +--> Redis Streams (async scheduling)
    |
    v
  MemScheduler (async ingestion, millisecond latency)
```

### MemCube

MemCube is the core abstraction. It is a composable unit of memory that supports:
- **Isolation** -- Each user or agent has a private MemCube
- **Sharing** -- MemCubes can be selectively shared across agents or tasks
- **Composition** -- Multiple MemCubes can be dynamically combined for a task
- **Evolution** -- Skills and knowledge within a MemCube evolve through use

### Skill Memory

The distinctive feature. When an agent successfully completes a task, the steps, tools used, and reasoning chain are stored as a "skill" in the MemCube. When a similar task arises later, the skill can be retrieved and reused, enabling:
- Cross-task skill transfer
- Skill evolution through refinement
- Task summarization with automatic improvement

## Memory Types

| Type | Analogy | Description |
|------|---------|-------------|
| **Plaintext** | Disk | Raw text, documents, conversation logs |
| **Activation** | RAM | Hot embeddings and cached representations for fast access |
| **Parametric** | CPU Cache | Knowledge internalized into model weights (fine-tuning) |
| **Skill** | Procedures | Reusable task execution patterns |
| **Tool** | Function Library | Tool usage traces for agent planning |
| **Persona** | Identity | User/agent personality and preference profiles |

## Storage Backends

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Vector DB** | Qdrant | Embedding storage and semantic search |
| **Graph DB** | Neo4j | Relationship mapping and knowledge graph |
| **Relational** | SQLite | Local metadata and lightweight deployments |
| **Cache/Queue** | Redis Streams | Asynchronous scheduling via MemScheduler |

## Retrieval Methods

1. **Semantic Search** -- Vector similarity via Qdrant embeddings
2. **Graph Traversal** -- Knowledge graph navigation via Neo4j
3. **Skill Matching** -- Find relevant past skills for current task
4. **Memory-Aware Chat** -- Retrieval integrated into conversation flow
5. **Multi-Modal Search** -- Images and charts (v2.0)

## Agent Integration

- **REST API** -- `POST /product/add`, `POST /product/search`
- **Python SDK** -- `pip install MemoryOS`
- **MemScheduler** -- Async ingestion with millisecond-level latency
- **Natural Language Feedback** -- Refine memories with corrections in plain language
- **Multi-Modal** -- Text, images, charts, tool traces, personas (v2.0)
- **Cloud + Self-Hosted** -- Both deployment options available
- **Supported Agents** -- moltbot, clawdbot, openclaw

```python
from memoryos import MemOS

mos = MemOS()

# Add knowledge
mos.add("The deployment pipeline uses GitHub Actions with staging -> production")

# Add a skill (cross-task reusable)
mos.add_skill(
    name="deploy_service",
    steps=["run tests", "build docker image", "push to registry", "update k8s"],
    tools=["pytest", "docker", "kubectl"],
    context="Standard microservice deployment procedure"
)

# Search memories
results = mos.search("How do we deploy?")
# Returns: deployment pipeline info + deploy_service skill

# Skill reuse in new context
skill = mos.get_skill("deploy_service")
# Agent can follow the skill steps for a new service deployment
```

## Benchmarks

MemOS does not prominently publish LoCoMo or LongMemEval benchmark scores. The project focuses on the operating system abstraction and skill memory capabilities rather than competitive retrieval benchmarks.

The v2.0 "Stardust" release (Dec 2025) introduced comprehensive KB support, memory feedback, multi-modal memory, and tool memory, but comparative performance data against other systems is limited.

## Strengths

- Unique "memory operating system" abstraction with MemCube composability
- Skill memory for cross-task reuse is a genuinely novel capability no other system offers
- Three-tier memory model (plaintext/activation/parametric) mirrors computer architecture well
- Multi-modal support (text, images, charts, tool traces)
- MemScheduler provides async ingestion with millisecond latency
- Natural language memory correction is a useful UX feature
- First to propose the "Memory OS" concept (May 2025 paper)
- Active development with v2.0 release

## Weaknesses

- No published standardized benchmark scores
- Complex architecture with 4 backend dependencies (Qdrant, Neo4j, SQLite, Redis)
- Parametric memory (model weight updates) requires significant infrastructure
- Skill memory quality depends on how well task execution is captured
- Relatively new concept; limited production deployment evidence
- MemCube composition rules may be complex to configure correctly
- No MCP server or CLI for IDE integration
- Documentation could be more comprehensive

## Usage Example

```python
from memoryos import MemOS

# Initialize with all backends
mos = MemOS(
    qdrant_url="http://localhost:6333",
    neo4j_url="bolt://localhost:7687",
    redis_url="redis://localhost:6379"
)

# Plaintext memory
mos.add("Alice is the tech lead. She prefers async reviews via GitHub PRs.")

# Skill memory -- capture a successful task execution
mos.add_skill(
    name="debug_memory_leak",
    steps=[
        "1. Check heap usage with pprof",
        "2. Identify top allocators",
        "3. Look for unclosed resources in hot paths",
        "4. Add defer/close statements",
        "5. Verify with load test"
    ],
    tools=["pprof", "go tool trace", "vegeta"],
    outcome="Reduced memory usage from 4GB to 800MB",
    context="Go microservice with goroutine leak in HTTP handler"
)

# Later: agent encounters a new memory leak
results = mos.search("How to fix memory leak in Go service?")
# Returns the debug_memory_leak skill with all steps and tools

# MemCube composition for a specific task
cube = mos.compose_cube(
    sources=["alice_preferences", "team_skills", "project_context"],
    task="Review Alice's PR for the payment service"
)
# Cube combines Alice's preferences (async review) + team skills
# (relevant debugging/deployment skills) + project context

# Feedback: refine a memory
mos.feedback("The deploy skill should include a rollback step after kubectl apply")
```
