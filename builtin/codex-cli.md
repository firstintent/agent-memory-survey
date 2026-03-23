# OpenAI Codex CLI Memory System

## Overview

Codex CLI uses the most sophisticated memory architecture among CLI-based coding agents: a two-phase pipeline where Phase 1 captures raw observations during task execution and Phase 2 consolidates them into structured knowledge via a dedicated sub-agent. Storage uses SQLite for runtime state and a hierarchy of markdown files plus a skills directory for persistent knowledge.

## Storage Locations

| Component | Path | Purpose |
|-----------|------|---------|
| SQLite state DB | `codex_home/state_<version>.sqlite` | Runtime state for agent jobs |
| Consolidated handbook | `memory/MEMORY.md` | Structured knowledge by task group |
| User profile | `memory/memory_summary.md` | Short profile + tips, injected into system prompt |
| Raw memories | `memory/raw_memories.md` | Merged Phase 1 raw observations, latest-first |
| Rollout summaries | `memory/rollout_summaries/` | Per-rollout summary files |
| Skills | `memory/skills/<skill-name>/SKILL.md` | Reusable procedural knowledge |
| System skills | `~/.codex/skills/.system/` | Built-in skills (plan, skill-creator) |
| User global skills | `~/.codex/skills/` | User-created global skills |
| Project instructions | `AGENTS.md` or `codex.md` (project root) | Human-authored project instructions |
| Config | `codex_home/config.yaml` | Agent configuration |

## Memory Types

### Phase 1: Raw Memory Capture

During task execution (a "rollout"), the agent captures observations:

- Commands that worked or failed
- Code patterns discovered
- Architecture insights
- User corrections and preferences

These are stored as `raw_memory` fields in the SQLite database and merged into `raw_memories.md`.

### Phase 2: Consolidation

A dedicated consolidation sub-agent reads Phase 1 outputs from SQLite and local filesystem artifacts, then updates:

- **MEMORY.md**: The main handbook, organized by task group. Structured, deduplicated, and prioritized.
- **memory_summary.md**: A compressed user profile and key tips. This is what gets injected into the system prompt for every interaction.
- **skills/**: Procedural knowledge extracted into reusable skill definitions.

### Skills Directory

Skills are the most distinctive feature. Each skill is a directory containing a `SKILL.md` file that describes a reusable procedure.

**Built-in system skills** (in `~/.codex/skills/.system/`):
- `plan` -- Planning methodology
- `skill-creator` -- Meta-skill for creating new skills

**User skills** can be placed in `~/.codex/skills/` (global) or `memory/skills/` (project-local). Codex discovers any directory in the tree with a `SKILL.md` file.

Skills encode procedural knowledge that goes beyond declarative facts -- they capture *how* to do things, not just *what* is true.

### Project Instructions

`AGENTS.md` (or `codex.md`) in the project root provides human-authored persistent instructions, analogous to `CLAUDE.md` or `.cursorrules`.

## Persistence Scope

- **SQLite state**: Persists across sessions. Versioned by schema (`state_<version>.sqlite`).
- **MEMORY.md / memory_summary.md**: Persist indefinitely. Updated by the consolidation pipeline.
- **Raw memories**: Persist as append-only log. Source material for consolidation.
- **Skills**: Persist indefinitely. Grow over time as the agent creates new skills.
- **Rollout summaries**: Persist per-rollout for historical reference.

## User Control

- `memory_summary.md` is directly editable (it's the injected context).
- `MEMORY.md` can be manually edited though the agent manages it.
- Skills can be created manually by adding `SKILL.md` files to the skills directory.
- Project instructions (`AGENTS.md`) are fully human-controlled.
- No built-in memory review UI -- file-system inspection required.

## Implementation Details

- **Two-phase architecture** separates capture from consolidation, avoiding the "growing pile of raw notes" problem that plagues simpler auto-memory systems.
- **SQLite backend** provides ACID guarantees for state management during concurrent or resumable operations.
- **Consolidation sub-agent** is a separate LLM invocation dedicated to organizing memory -- not a side-effect of task execution.
- **memory_summary.md** is the only file injected into the system prompt, keeping token usage bounded. Full `MEMORY.md` is available for deeper retrieval.
- **Skill discovery** is filesystem-based: any `SKILL.md` in the skills tree is automatically available.

## Strengths

- **Two-phase pipeline is genuinely novel**: Separation of raw capture and structured consolidation produces higher-quality persistent memory than raw append-only logs.
- **Skills system**: Procedural knowledge capture goes beyond what declarative memory files offer. The agent can learn *how* to do things.
- **Bounded prompt injection**: Only `memory_summary.md` is injected, keeping context window usage predictable.
- **SQLite reliability**: ACID-compliant storage with versioned schema for state management.
- **Self-improving**: The skill-creator meta-skill enables autonomous growth of the skills library.
- **Local-first**: All storage is on the local filesystem. No cloud dependency.

## Weaknesses

- **Complexity**: The two-phase pipeline with SQLite + markdown + skills is significantly more complex than competitors' approaches. More moving parts to debug.
- **Opaque consolidation**: The sub-agent's decisions about what to consolidate and how to organize `MEMORY.md` aren't transparent to users.
- **No approval workflow**: Memories and skills are created/updated without user review.
- **No semantic search**: Despite the sophistication, retrieval is still file-based, not vector-indexed.
- **Ecosystem lock-in**: Skills are Codex-specific; they don't port to other agents.
- **Consolidation cost**: Phase 2 requires additional LLM calls, adding latency and token cost.

## Sources

- [Memory System (DeepWiki)](https://deepwiki.com/openai/codex/3.7-memory-system)
- [Agent Skills (OpenAI Developers)](https://developers.openai.com/codex/skills)
- [Configuration Reference (OpenAI Developers)](https://developers.openai.com/codex/config-reference)
- [Skills in OpenAI Codex (Blog)](https://blog.fsck.com/2025/12/19/codex-skills/)
- [CLI Features (OpenAI Developers)](https://developers.openai.com/codex/cli/features)
