# claude-subconscious

## GitHub URL

https://github.com/letta-ai/claude-subconscious

## Summary

Claude Subconscious is a Letta-powered background agent that gives Claude Code persistent memory across sessions. It works as a "sidecar" Letta agent that observes Claude Code session transcripts, accumulates context and patterns over time, and injects guidance back into future sessions via the CLAUDE.md file. Built by the Letta team, it offers one-click install and positions itself as a way to extend the closed-source Claude Code agent with open, programmable memory.

## Architecture

The system has three main components:

1. **Letta Sidecar Agent**: A full Letta agent that runs alongside Claude Code. It has its own memory blocks (user preferences, project context, session patterns) and tool access via the Letta Code SDK.
2. **Transcript Relay**: Each time Claude Code stops (completes a response), the session transcript is sent to the Letta sidecar agent for processing.
3. **CLAUDE.md Injection**: The Letta agent's memory blocks are appended to the CLAUDE.md file on session start. Claude Code reads CLAUDE.md as part of its system prompt, so it sees whatever the subconscious has prepared.
4. **Guidance Block**: A curated communication channel from the subconscious to Claude Code. Rather than raw message injection, the subconscious writes structured guidance that Claude Code consumes.

The processing is **asynchronous** -- the subconscious receives transcripts after the fact and prepares context for future sessions, not the current one.

Using Letta's Conversations feature, a single subconscious agent can serve **multiple Claude Code sessions in parallel** with shared memory across all of them.

## How It Works

1. User starts a Claude Code session. The subconscious agent's memory blocks are loaded into CLAUDE.md.
2. Claude Code runs normally, seeing the injected memory as part of its context.
3. When Claude Code finishes a response, the transcript is relayed to the Letta sidecar.
4. The sidecar processes the transcript asynchronously: updates its memory blocks, identifies patterns, and optionally takes actions (open GitHub issues, search the web, message Slack/Discord).
5. On the next session start, the updated memory blocks are injected into CLAUDE.md again.

The subconscious has full tool access via the Letta Code SDK, meaning it can read files, search the web, and explore the codebase while processing transcripts -- it is not limited to passive memory operations.

## Strengths

- **One-click install**: Low friction to get started, important for adoption.
- **Persistent cross-session memory**: Solves the fundamental problem of Claude Code forgetting everything between sessions.
- **Multi-session shared memory**: A single agent serves all parallel Claude Code sessions, keeping them in sync.
- **Extensible via tools**: The subconscious can do more than just memory -- it can take autonomous actions (file tickets, message channels, do research).
- **Non-invasive**: Works through CLAUDE.md injection, requiring no modifications to Claude Code itself.
- **Backed by Letta**: Built on a mature agent framework with proper memory management primitives.

## Weaknesses

- **Asynchronous only**: Memory updates apply to the next session, not the current one. The subconscious cannot intervene in real-time.
- **CLAUDE.md size limits**: Only the first 200 lines of CLAUDE.md (including the memory/ directory) load into Claude Code's system prompt. Memory must be carefully curated to fit.
- **Letta dependency**: Requires running a Letta server/agent, adding infrastructure complexity.
- **Closed-loop risk**: The subconscious can take autonomous actions (open tickets, message Slack) which could produce unwanted side effects if not carefully configured.
- **Opaque memory curation**: The Letta agent decides what to remember and what guidance to provide, and this decision process may not be transparent or controllable by the user.
- **Single point of failure**: If the Letta sidecar crashes or falls behind, memory injection stops working silently.
