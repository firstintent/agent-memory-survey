# SuperLocalMemory V3

## Project Info

| Field | Value |
|-------|-------|
| **GitHub** | [qualixar/superlocalmemory](https://github.com/qualixar/superlocalmemory) |
| **Stars** | 38 |
| **License** | MIT |
| **Language** | Python |
| **Status** | Active, research-backed |
| **Paper** | [arXiv:2603.14588](https://arxiv.org/abs/2603.14588) (V3), [arXiv:2603.02240](https://arxiv.org/abs/2603.02240) (V2) |
| **Website** | [superlocalmemory.com](https://www.superlocalmemory.com/) |

## Summary

SuperLocalMemory V3 is a local-only AI memory system built on information geometry, algebraic topology, and stochastic dynamics. It is the first agent memory system with mathematical guarantees for retrieval (Fisher-Rao metric), consistency (sheaf cohomology), and memory lifecycle (Langevin dynamics). The system achieves 74.8% on LoCoMo with zero cloud dependency -- outperforming Mem0 by 16 percentage points without a single API call. Designed for GDPR/HIPAA/EU AI Act compliance, all data stays on-device.

## Architecture

```
Input (text, conversations)
    |
    v
  4-Channel Hybrid Retrieval
    +--> Semantic (Fisher-Rao information geometry)
    +--> BM25 (lexical keyword matching)
    +--> Entity Graph (relationship traversal)
    +--> Temporal (time-aware filtering)
    |
    v
  RRF Fusion + Cross-Encoder Reranking
    |
    v
  SQLite (local persistent storage)
    |
  Three Operating Modes:
    Mode A: Zero-cloud (default) -- no APIs, no LLM
    Mode B: Local Ollama -- local LLM augmentation
    Mode C: Cloud LLM -- optional cloud enhancement
```

### Mathematical Foundations

1. **Fisher-Rao Retrieval Metric** -- Retrieval distance derived from the Fisher information structure of diagonal Gaussian families. This provides a principled geometric measure of memory similarity rather than ad-hoc cosine similarity.
2. **Sheaf Cohomology Consistency** -- Memory coherence modeled as a cellular sheaf. Non-trivial first cohomology classes correspond precisely to irreconcilable contradictions across memory contexts. The system can provably detect inconsistencies.
3. **Langevin Memory Lifecycle** -- Memory decay/consolidation formulated as Riemannian Langevin dynamics with proven convergence via Fokker-Planck equation. Memories naturally consolidate or fade based on mathematical dynamics.

## Memory Types

| Type | Description |
|------|-------------|
| **Semantic** | Dense embeddings with Fisher-Rao geometric distance |
| **Lexical** | BM25-indexed keyword representations |
| **Entity** | Graph-structured entity relationships |
| **Temporal** | Time-stamped memories with decay dynamics |

## Storage Backends

| Component | Technology |
|-----------|------------|
| **Primary** | SQLite (local, on-device) |
| **Profiles** | Isolated memory spaces per user/project |
| **Audit** | SHA-256 immutable audit chain |

No external databases. No cloud storage. Everything in local SQLite.

## Retrieval Methods

1. **Fisher-Rao Semantic** -- Information-geometric distance on Gaussian manifold
2. **BM25 Lexical** -- Standard sparse keyword matching
3. **Entity Graph Traversal** -- Relationship-following across entity graph
4. **Temporal Filtering** -- Time-window queries with Langevin decay weighting
5. **RRF Fusion** -- Reciprocal rank fusion across all 4 channels
6. **Cross-Encoder Reranking** -- Final reranking pass for precision

Mathematical retrieval layers contribute +12.7 percentage points on average.

## Agent Integration

**MCP Server** -- 24 MCP tools + 6 MCP resources for IDE integration:
- Claude Code, Cursor, Windsurf, VS Code Copilot, Continue, Cody
- ChatGPT Desktop, Gemini CLI, JetBrains, Zed
- 17+ AI tools total

**Agent-Native CLI** -- Structured JSON output for programmatic use:
- `remember`, `recall`, `forget`, `trace`, `status`, `health`
- `mode`, `setup`, `warmup`, `migrate`, `dashboard`, `mcp`
- `connect`, `profile`

```bash
# CLI usage
slm remember "User prefers dark mode and uses neovim"
slm recall "What editor does the user prefer?"
slm mode a  # Switch to zero-cloud mode
slm status  # Show memory statistics
```

## Benchmarks

| Benchmark | Mode A (Local) | Mode C (Cloud) | Notes |
|-----------|---------------|----------------|-------|
| **LoCoMo Overall** | 74.8% | 87.7% | Mode A is zero-cloud |
| **Open-Domain Questions** | 85.0% | -- | Highest of any system evaluated |
| **vs. Mem0** | +16pp | -- | Without any API calls |
| **Math Layer Contribution** | +12.7pp | -- | Fisher-Rao + sheaf + Langevin |

## Compliance Features

| Standard | Support |
|----------|---------|
| **GDPR** | Article 15/17 export + complete erasure |
| **HIPAA** | Local-only storage, no data transmission |
| **EU AI Act** | Audit trails, provenance tracking |
| **Audit** | SHA-256 tamper-proof chain |
| **Access Control** | ABAC policy enforcement |
| **Provenance** | Full data lineage tracking |

## Strengths

- 74.8% LoCoMo with zero cloud dependency is remarkable
- Mathematical foundations provide provable guarantees (not just heuristics)
- GDPR/HIPAA/EU AI Act compliance by architecture, not policy
- 24 MCP tools with broadest IDE compatibility in the space
- Three operating modes (zero-cloud, local LLM, cloud) for flexibility
- SQLite-only storage eliminates all infrastructure dependencies
- Sheaf cohomology contradiction detection is a unique capability
- Immutable audit trails with SHA-256

## Weaknesses

- Very small community (38 stars) limits ecosystem and support
- Mathematical complexity may hinder adoption and contribution
- No multimodal support (text only)
- SQLite scalability limits for very large memory stores
- No multi-agent memory sharing
- Relatively new project with limited production deployment evidence
- Mathematical guarantees may be overkill for simple use cases
- Mode A accuracy (74.8%) still trails cloud-augmented systems

## Usage Example

```bash
# Install and configure
pip install superlocalmemory
slm setup

# Store memories (Mode A: zero-cloud)
slm remember "Alice is the project lead for the redesign sprint"
slm remember "Sprint deadline is March 28, review meeting on March 25"
slm remember "Alice prefers async communication via Slack"

# Recall with 4-channel hybrid retrieval
slm recall "When is the sprint deadline?"
# {
#   "memories": [
#     {"content": "Sprint deadline is March 28, review meeting on March 25",
#      "score": 0.94, "channels": ["semantic", "temporal", "lexical"]}
#   ],
#   "mode": "A", "latency_ms": 12
# }

# Compliance operations
slm trace "Alice"  # Show all memories mentioning Alice with provenance
slm forget --user "alice" --gdpr  # GDPR Article 17 erasure

# MCP integration (add to Claude Code settings)
# "superlocalmemory": {"command": "slm", "args": ["mcp"]}
```
