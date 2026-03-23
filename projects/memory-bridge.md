# memory-bridge

## GitHub URL

No exact repository named "memory-bridge" was found. The concept of bidirectional memory sync between Codex CLI and Claude Code is implemented by several projects in the ecosystem. The closest matches are:

- **codex-claude-bridge**: https://github.com/abhishekgahlot2/codex-claude-bridge -- Bidirectional bridge between Claude Code and OpenAI Codex CLI using Claude Code Channels.
- **AutoMemoryBridge (claude-flow)**: https://github.com/ruvnet/claude-flow/issues/1102 -- Bidirectional sync between Claude Code's auto memory and claude-flow's AgentDB.
- **memorix**: https://gitea.com/Fromsko/memorix (also https://www.npmjs.com/package/memorix) -- Cross-agent memory bridge across Cursor, Windsurf, Claude Code, Codex, Copilot, Kiro via MCP.

The X post from March 18, 2026 may refer to one of these projects or a project that is not yet publicly indexed.

## Summary

"memory-bridge" refers to the concept (and likely a specific project) of bidirectional memory synchronization between Codex CLI and Claude Code. The goal is to ensure that context, preferences, and learned information persist and stay synchronized across different AI coding agents, so switching between tools does not mean losing accumulated project knowledge. The design is intended to be memory-system-agnostic, working with any underlying storage backend.

## Architecture (Based on Ecosystem Projects)

The general pattern across bidirectional memory bridge implementations:

1. **Shared Memory Layer**: A common storage backend (SQLite, LanceDB, or markdown files in `~/.claude/projects/*/memory/`) that both agents can read from and write to.
2. **Agent Adapters**: Per-agent integration that hooks into each tool's session lifecycle (Claude Code hooks, Codex CLI hooks/tool calls).
3. **Sync Protocol**: Bidirectional sync that propagates memory changes from one agent to the other, either through file watching, MCP tools, or explicit sync commands.
4. **Memory Format**: Typically markdown-based or structured JSON, with semantic search capabilities for retrieval.

In the case of codex-claude-bridge specifically, it uses Claude Code Channels as the push mechanism on Claude's side and a blocking MCP tool call on Codex's side.

## How It Works

1. Agent A (e.g., Claude Code) makes a decision or learns something during a session.
2. The memory bridge captures this as a memory entry and writes it to the shared store.
3. Agent B (e.g., Codex CLI) starts a session or receives a task.
4. The bridge injects relevant memories from the shared store into Agent B's context.
5. Any new learnings from Agent B are written back, available for Agent A's next session.

This creates a unified memory across heterogeneous agents.

## Strengths

- **Agent-agnostic**: Works across different AI coding tools rather than locking into one ecosystem.
- **Prevents context loss**: Switching between Claude Code and Codex CLI (or other tools) no longer means starting from scratch.
- **Memory-system-agnostic**: Claimed to work with any underlying memory system, providing flexibility.
- **Practical need**: Addresses a real pain point for developers who use multiple AI coding assistants.

## Weaknesses

- **No confirmed public repo**: The specific "memory-bridge" project from the March 18 X post could not be located, making it difficult to assess implementation quality.
- **Sync conflicts**: Bidirectional sync is inherently complex -- concurrent writes from different agents could create conflicts or stale data.
- **Integration fragility**: Depends on stable APIs/hooks from both Claude Code and Codex CLI, which are rapidly evolving and could break adapters.
- **Memory format alignment**: Different agents may represent knowledge differently, requiring lossy translation between formats.
- **Ecosystem fragmentation**: Multiple competing projects (codex-claude-bridge, memorix, AutoMemoryBridge) suggests no dominant standard has emerged yet.
