# Augment Code Memory System

## Overview

Augment Code features a Memory Review system that surfaces agent-created memories for user approval before they are persisted. This is the only major AI coding agent with an explicit inline approval workflow for memories. Augment is designed for large codebases and emphasizes context-aware assistance with deep codebase understanding.

## Storage Location

| Component | Location | Scope |
|-----------|----------|-------|
| Agent memories | Augment cloud | Per-project / per-user |
| Memory review queue | Inline in chat panel | Per-session (pending memories) |

Exact local storage paths are not publicly documented. Augment operates as a cloud-connected IDE extension.

## Memory Types

### Agent Memories (with Approval Workflow)

Augment's agent creates memories during coding sessions -- observations about code patterns, user preferences, architectural decisions, and project conventions. The key differentiator is the **Memory Review** feature:

1. When a new memory is created, a **"X Pending Memory" button** appears in the turn summary within the chat panel.
2. Clicking it reveals the memory content inline.
3. The user can **approve**, **edit**, or **discard** each memory.
4. Nothing is stored without user sign-off.

Memory content examples:
- Code style patterns observed
- Architectural conventions
- User workflow preferences
- Project-specific rules and constraints

### Context-Aware Retrieval

Augment indexes large codebases and uses deep context retrieval (not just memory files) to understand cross-file relationships, dependencies, and patterns. This codebase understanding supplements the explicit memory system.

## Persistence Scope

- **Approved memories**: Persist across sessions. Available for future interactions.
- **Pending memories**: Exist only during the session until approved or discarded.
- **Discarded memories**: Gone permanently.
- **Codebase index**: Maintained by Augment for retrieval. Separate from explicit memories.

## User Control

- **Inline review**: Approve, edit, or discard every memory directly in the chat panel.
- **No silent memory creation**: Nothing persists without explicit user action.
- **Edit before saving**: Users can modify memory content before approving.
- **Per-memory granularity**: Each memory is reviewed individually, not in bulk.

This is the strongest user control model among AI coding agents. Compared to:
- Claude Code's auto-memory (silent, review after the fact)
- ChatGPT's saved memories (silent, delete after the fact)
- Windsurf's cascade memories (silent, filesystem inspection required)

## Implementation Details

- Memory Review is integrated directly into the chat panel UX, not a separate settings page.
- The review flow is part of the natural chat loop: code -> agent creates memory -> user sees pending indicator -> reviews inline -> continues coding.
- Augment's codebase indexing supports large enterprise codebases (marketed for repos with millions of lines of code).
- The memory system works alongside Augment's deep context engine, which understands code relationships beyond what's in explicit memories.

## Strengths

- **Approval workflow is best-in-class**: The only agent that shows you every memory before it's saved and lets you edit or discard.
- **Inline UX**: Memory review happens in the chat flow, not in a separate admin panel. Minimal friction.
- **No surprise memories**: Users always know exactly what the agent remembers.
- **Edit capability**: Not just approve/reject -- users can refine memory content before saving.
- **Large codebase support**: Deep indexing handles enterprise-scale repos.

## Weaknesses

- **Approval friction**: Every memory requires a user action. For power users who trust the agent, this adds overhead that Claude Code or Windsurf avoid.
- **Cloud-dependent**: Memory storage is on Augment's servers. No local file artifacts to version-control or inspect.
- **Limited public documentation**: Technical details of storage format, retrieval mechanism, and memory organization are not well-documented.
- **No file-system memory artifacts**: Unlike CLAUDE.md or .cursorrules, there's nothing in the repo for team sharing.
- **Newer product**: Smaller community and fewer third-party integrations compared to Cursor or Copilot.
- **No memory export**: Memories are locked in Augment's system.

## Sources

- [How We Built Memory Review (Augment Code Blog)](https://www.augmentcode.com/blog/how-we-built-memory-review)
- [Agent Memory Review Changelog (Augment Code)](https://www.augmentcode.com/changelog/memory-review)
- [Augment Code IDE Agents (Product Page)](https://www.augmentcode.com/product/ide-agents)
- [Augment Code Review 2026 (VibecodedThis)](https://www.vibecodedthis.com/reviews/augment-code-review-2026/)
