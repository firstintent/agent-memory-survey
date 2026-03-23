# Windsurf (Codeium) Memory System

## Overview

Windsurf's Cascade agent uses a multi-layered context system: auto-generated memories stored locally, `.windsurfrules` instruction files, and a 5-source context assembly pipeline. Memories are created automatically during conversation and persisted per-workspace.

## Storage Locations

| Layer | Path | Author |
|-------|------|--------|
| Auto-generated memories | `~/.codeium/windsurf/memories/` | Agent (per-workspace) |
| Project rules | `.windsurfrules` (project root) | Human |
| Global rules | Windsurf settings | Human |

## Memory Types

### Cascade Memories (Auto-Generated)

During conversation, Cascade automatically generates and stores memories when it encounters context it deems useful. Users can also explicitly prompt Cascade to "create a memory of ..." at any time.

Typical memory content:
- Project architecture patterns
- Coding conventions observed
- Build/deployment configurations
- User preferences and workflow habits
- Framework-specific patterns

Memories are scoped to the workspace they were created in. They are stored locally and **do not consume credits**.

### .windsurfrules (Project Instructions)

A markdown file in the project root that tells Cascade about your stack, conventions, and boundaries. Analogous to `.cursorrules` or `CLAUDE.md`. Typically version-controlled and committed to git for team-wide consistency.

Content typically includes:
- Technology stack declarations
- Coding style rules
- Forbidden patterns or libraries
- Project structure conventions

### Global Rules

User-level preferences configured in Windsurf settings. Apply across all workspaces.

## Context Assembly Pipeline

Cascade assembles context from 5 sources on every interaction, in priority order:

1. **Rules** (`.windsurfrules` + global rules)
2. **Memories** (workspace-scoped auto-memories)
3. **Open files** (currently visible editors)
4. **Indexed retrieval** (codebase index search)
5. **Recent actions** (what was just done in the editor)

## Persistence Scope

- **Cascade memories**: Persist locally in `~/.codeium/windsurf/memories/`. Survive across sessions. Workspace-scoped.
- **.windsurfrules**: Persists in the repo. Shared via git.
- **Global rules**: Persist in Windsurf settings. Local to the installation.
- **Learned behavior**: Windsurf reports noticeably improved suggestions after ~48 hours of use as it accumulates workspace memories.

## User Control

- Users can prompt Cascade to create specific memories.
- Users can view and delete memories (stored as local files).
- `.windsurfrules` is directly editable.
- No built-in memory approval UI -- memories are created silently.
- No built-in memory review dashboard.

## Implementation Details

- Memories are local-only. No cloud sync between machines.
- Memory creation and retrieval do not consume API credits.
- The context engine selectively surfaces relevant memories per interaction rather than loading all memories.
- Codebase indexing supports the retrieval layer but is separate from the memory system.
- Memories are workspace-scoped, preventing cross-project contamination.

## Strengths

- **Zero-cost memories**: Auto-generated memories don't consume credits, encouraging liberal use.
- **5-source context pipeline**: Holistic context assembly goes beyond just memory files.
- **Workspace scoping**: Clean separation between projects prevents memory pollution.
- **Implicit learning**: ~48 hours of use yields noticeably better context without manual setup.
- **Simple rules file**: `.windsurfrules` is easy to create and share.
- **Explicit memory creation**: Users can force important memories via natural language prompts.

## Weaknesses

- **No memory review/approval UI**: Memories are created silently; users must manually inspect the filesystem to audit them.
- **Local-only storage**: Memories don't sync across machines. Team members don't share auto-memories.
- **No memory expiration**: Unlike GitHub Copilot's 28-day TTL, memories persist indefinitely. Manual cleanup required.
- **Single rules file**: `.windsurfrules` is one monolithic file, unlike Cursor's multi-file `.cursor/rules/` with activation scopes.
- **Opaque retrieval**: Users can't see which memories were used in a given interaction.
- **No version control for memories**: Auto-memories in `~/.codeium/` are outside the repo.

## Sources

- [Cascade Memories (Windsurf Docs)](https://docs.windsurf.com/windsurf/cascade/memories)
- [Windsurf Rules & Workflows Best Practices](https://www.paulmduvall.com/using-windsurf-rules-workflows-and-memories/)
- [Understanding Windsurf's Memories System (Arsturn)](https://www.arsturn.com/blog/understanding-windsurf-memories-system-persistent-context)
- [Windsurf Flow Context Engine (Markaicode)](https://markaicode.com/windsurf-flow-context-engine/)
