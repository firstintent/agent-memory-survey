# GitHub Copilot Memory System

## Overview

GitHub Copilot features an agentic memory system where the Copilot agent autonomously discovers, stores, and retrieves facts about a repository. Memories are cloud-hosted, repo-scoped, and shared across all users with access. As of March 2026, memory is on by default for Pro and Pro+ users in public preview.

## Storage Location

| Component | Location | Scope |
|-----------|----------|-------|
| Copilot Memory | GitHub cloud (per-repository) | Repository-scoped |
| Custom instructions | `.github/copilot-instructions.md` | Repository-scoped, version-controlled |
| VS Code instructions | `.github/copilot-instructions.md` or settings | User/workspace |

Memories are **not** stored locally. They live on GitHub's servers, tied to the repository.

## Memory Types

### Agentic Memory (Auto-Generated)

The Copilot agent discovers and stores facts during normal operation:

- Coding conventions (naming, formatting, patterns)
- Architectural patterns (project structure, module boundaries)
- Cross-file dependencies (critical imports, API contracts)
- Build/test configurations

Memories are created by the agent without explicit user action. The agent validates memories against the current codebase before applying them, so stale context is filtered out.

### Custom Instructions (Human-Authored)

The `.github/copilot-instructions.md` file provides persistent human-written instructions, similar to Cursor's rules or Claude Code's `CLAUDE.md`. These are version-controlled and shared via git.

## Persistence Scope

- **Agentic memories**: Automatically expire after **28 days**. No manual renewal. If the pattern is still present in the codebase, the agent will rediscover it.
- **Custom instructions**: Persist indefinitely in the repo.
- **Cross-agent sharing**: Memories discovered by the coding agent are available to code review, and vice versa. Also available to Copilot CLI.
- **Cross-user sharing**: All users with Copilot Memory access on a repo share the same memory pool.

## User Control

- **Repository owners** can review and delete stored memories: Repository Settings > Copilot > Memory.
- **Organization admins** can enable/disable Copilot Memory at the org level.
- Individual users cannot create memories manually -- the agent decides what to store.
- No per-user memory isolation within a repo (all memories are shared).
- Memories can be deleted individually through the settings UI.

## Implementation Details

- Memories are validated against the current codebase before injection. If the codebase has changed and the memory no longer applies, it's not used.
- The 28-day TTL prevents memory staleness without manual cleanup.
- Memory creation is triggered during agent interactions (coding, review, CLI usage) -- not by background analysis.
- No vector database or embeddings exposed to users; the retrieval mechanism is opaque.
- Supported across VS Code, GitHub.com, and Copilot CLI.

## Strengths

- **Zero-maintenance**: Automatic discovery, validation, and expiration. Users don't need to curate memories.
- **Cross-agent coherence**: Coding agent and review agent share the same knowledge, reducing inconsistency.
- **Team-shared by default**: One user's coding session benefits the entire team's future interactions.
- **Staleness prevention**: 28-day TTL + codebase validation ensures memories stay relevant.
- **No local storage**: Works identically across machines and environments.

## Weaknesses

- **Opaque memory creation**: Users can't directly create or prioritize memories. The agent decides autonomously.
- **28-day expiration is aggressive**: Long-lived architectural decisions may need to be rediscovered repeatedly. Custom instructions file is the workaround.
- **No user-level memory**: All memories are repo-scoped. Personal workflow preferences require the instructions file.
- **Cloud-only**: Requires GitHub connectivity. No offline access to memories.
- **No approval workflow**: Memories are created silently. Users can only review and delete after the fact.
- **Limited to Pro/Pro+**: Free-tier users don't have access to agentic memory (as of March 2026).
- **No cross-repo memory**: Knowledge discovered in one repo doesn't transfer to related repos.

## Sources

- [About Agentic Memory for GitHub Copilot (Docs)](https://docs.github.com/en/copilot/concepts/agents/copilot-memory)
- [Building an Agentic Memory System for GitHub Copilot (Blog)](https://github.blog/ai-and-ml/github-copilot/building-an-agentic-memory-system-for-github-copilot/)
- [Copilot Memory On by Default (Changelog)](https://github.blog/changelog/2026-03-04-copilot-memory-now-on-by-default-for-pro-and-pro-users-in-public-preview/)
- [Managing and Curating Copilot Memory (Docs)](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/copilot-memory)
- [Memory in VS Code Agents](https://code.visualstudio.com/docs/copilot/agents/memory)
