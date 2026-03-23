# Claude Code Memory System

## Overview

Claude Code uses a layered markdown-file memory system: human-authored `CLAUDE.md` instruction files plus agent-authored `MEMORY.md` auto-memory files. Both are plain text, loaded into the system prompt at session start.

## Storage Locations

| Layer | Path | Author |
|-------|------|--------|
| Project root | `./CLAUDE.md` | Human |
| Project subdirectories | `<subdir>/CLAUDE.md` | Human |
| User global | `~/.claude/CLAUDE.md` | Human |
| Enterprise/org | Set via admin policy | Human |
| Auto-memory (per-project) | `~/.claude/projects/<path-hash>/memory/MEMORY.md` | Agent |
| Auto-memory (global) | `~/.claude/memory/MEMORY.md` | Agent |

## Memory Types

### CLAUDE.md (Instruction Memory)

Human-written markdown files that act as persistent system instructions. Content is arbitrary but typically includes:

- Build and test commands
- Code style / linting rules
- Architecture constraints
- Framework and library preferences
- Workflow instructions (e.g., "always run tests before committing")

**Loading hierarchy:** Global user file -> project root file -> subdirectory files (loaded when those directories are referenced). Project-level instructions override global ones where they conflict.

### MEMORY.md (Auto-Memory)

Agent-written notes that Claude saves for itself based on corrections, discoveries, and preferences observed during sessions. Requires Claude Code v2.1.59+; on by default.

Typical auto-memory content:

- Build commands that work (`npm run build -- --no-cache`)
- Debugging insights ("the test suite requires `DATABASE_URL` to be set")
- Architecture notes ("services/ uses hexagonal architecture")
- User preferences ("user prefers explicit types over inference")
- Workflow habits

Auto-memory is stored per-project (keyed by workspace path hash) and globally. The agent appends to `MEMORY.md` during sessions; it does not delete entries on its own.

## Persistence Scope

- **CLAUDE.md**: Persists indefinitely in the filesystem. Version-controlled when committed to git. Shared across all users who clone the repo.
- **MEMORY.md**: Persists in `~/.claude/` across sessions. Local to the machine (not version-controlled by default). Survives Claude Code upgrades.
- **Session context**: Not persisted. Each session starts fresh, reading only from the above files.

## User Control

- Users can directly edit or delete any `CLAUDE.md` or `MEMORY.md` file.
- `/memory` command in Claude Code opens the relevant memory file for editing.
- Auto-memory can be disabled via settings (`--disable-auto-memory` flag or config).
- No approval workflow -- auto-memories are written silently. Users must review manually.
- Memory files can be `.gitignore`d selectively (e.g., commit `CLAUDE.md`, ignore personal `MEMORY.md`).

## Implementation Details

- All memory files are injected into the system prompt at session start, consuming context window tokens.
- Subdirectory `CLAUDE.md` files are loaded lazily -- only when files in that subtree are referenced.
- There is no semantic search or vector indexing; the entire file is loaded verbatim.
- No deduplication or summarization is performed automatically on `MEMORY.md`.
- The `#` import syntax allows `CLAUDE.md` to reference other files, enabling modular rule organization.

## Strengths

- **Transparent and auditable**: Everything is plain text in known locations. No opaque database.
- **Version-controllable**: `CLAUDE.md` files travel with the repo, giving teams shared agent behavior.
- **Zero-config auto-memory**: Agent learns preferences without manual setup.
- **Hierarchical override**: Global -> project -> subdirectory layering handles multi-project workflows cleanly.
- **No external dependencies**: No database, no cloud sync, no embeddings runtime.

## Weaknesses

- **No semantic retrieval**: Large memory files waste context tokens on irrelevant entries. Everything is loaded linearly.
- **Manual curation required**: `MEMORY.md` grows monotonically. Stale or contradictory entries accumulate without cleanup.
- **No approval flow**: Auto-memories are saved silently; users may not realize incorrect information was persisted.
- **Token budget pressure**: Memory files compete with code context for the context window. Large `CLAUDE.md` files reduce available space for actual code.
- **Machine-local auto-memory**: `MEMORY.md` doesn't sync across machines or team members without manual effort.
- **No cross-session summarization**: Unlike systems with compression pipelines, memories are raw appended text.

## Sources

- [Claude Code Memory Docs](https://code.claude.com/docs/en/memory)
- [SFEIR Institute - CLAUDE.md Deep Dive](https://institute.sfeir.com/en/claude-code/claude-code-memory-system-claude-md/deep-dive/)
- [Claude Code MEMORY.md Guide](https://levelup.gitconnected.com/claude-code-memory-md-everything-you-need-to-know-how-to-get-started-8ac99e161153)
