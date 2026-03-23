# Cursor Memory System

## Overview

Cursor uses a rule-based memory system with `.cursor/rules/` directory files, legacy `.cursorrules` root file, and a community-driven "Memory Bank" methodology built on custom instructions. There is no built-in auto-memory -- persistence is achieved through manually or community-crafted rule files and structured documentation workflows.

## Storage Locations

| Layer | Path | Scope |
|-------|------|-------|
| Project rules (new) | `.cursor/rules/*.mdc` | Per-project, version-controlled |
| Legacy project rules | `.cursorrules` (project root) | Per-project, single file |
| User rules (global) | Cursor Settings > General > Rules for AI | Per-user, all projects |
| Memory Bank (community) | `memory-bank/` directory in project | Per-project, version-controlled |

## Memory Types

### Project Rules (`.cursor/rules/`)

Markdown files (`.mdc` extension) stored in the project repo. Each file can have frontmatter specifying activation scope. Multiple files allow splitting rules by concern (e.g., `frontend.mdc`, `testing.mdc`, `api.mdc`).

**Activation scopes:**

- **Always**: Loaded into every request. Use sparingly -- consumes context tokens constantly.
- **Auto Attached**: Triggered automatically when editing files matching a glob pattern (e.g., `*.tsx`).
- **Agent Requested**: The agent reads the rule description and decides whether to apply it based on the current task.
- **Manual**: User explicitly adds the rule to a chat session.

**Priority:** Project rules override user/global rules on conflict.

### Legacy `.cursorrules`

Single file in project root. All rules in one monolithic block. Still supported but officially deprecated in favor of `.cursor/rules/`. No activation scoping -- always loaded.

### User Rules (Global)

Personal preferences set in Cursor settings UI. Apply to all projects. Lower priority than project rules. Not stored in the filesystem as a standalone file.

### Memory Bank (Community Methodology)

Not a built-in Cursor feature but a widely adopted community pattern. Uses custom instructions to tell Cursor to maintain a `memory-bank/` directory with structured markdown files:

- `projectbrief.md` -- Core requirements and goals
- `activeContext.md` -- Current work focus, recent changes, next steps
- `systemPatterns.md` -- Architecture, design patterns, component relationships
- `techContext.md` -- Technology stack, dependencies, tooling
- `progress.md` -- What works, what's left, blockers

The Memory Bank is typically paired with a structured workflow system using custom modes (VAN, PLAN, CREATIVE, IMPLEMENT, REFLECT, ARCHIVE) that load only relevant rules per phase, reducing token usage by ~70% compared to loading everything at once.

## Persistence Scope

- **Project rules**: Persist in the repo. Shared via git.
- **User rules**: Persist in Cursor's settings store. Local to the user's Cursor installation.
- **Memory Bank**: Persist as markdown files in the repo. Must be manually maintained or maintained via custom instructions.
- **Chat context**: Not persisted across sessions. Each new chat starts with rules only.

## User Control

- Users create, edit, and delete rule files directly in `.cursor/rules/`.
- Activation scopes give fine-grained control over when rules apply.
- No automatic memory creation -- all rules are human-authored.
- Memory Bank methodology requires explicit setup and maintenance (or reliance on agent self-updating via custom instructions).
- No built-in memory review or approval UI.

## Implementation Details

- Rules are injected as system prompt content, consuming context window tokens.
- `.mdc` files support frontmatter for metadata (description, activation scope, glob patterns).
- Nested rules: `.cursor/rules/` files in subdirectories auto-attach when files in that directory are referenced.
- No vector database, no embeddings, no semantic search.
- The Memory Bank approach uses plan/act mode separation to prevent the agent from making changes during planning phases.

## Strengths

- **Granular activation scoping**: Auto-attached and agent-requested rules reduce token waste vs. always-on approaches.
- **Version-controllable**: `.cursor/rules/` files are repo-native, enabling team-wide shared behavior.
- **Modular organization**: Multiple `.mdc` files per project allow clean separation of concerns.
- **Community ecosystem**: Extensive shared rule libraries and Memory Bank templates available.
- **Phase-based workflows**: Memory Bank methodology's hierarchical rule-loading is genuinely innovative for token efficiency.

## Weaknesses

- **No built-in auto-memory**: Cursor does not learn from sessions automatically. Everything must be manually authored or maintained via prompts.
- **Memory Bank is DIY**: The most powerful memory pattern requires significant setup and is not officially supported -- it's a community hack.
- **No cross-session continuity by default**: Without Memory Bank or similar, every new chat is stateless.
- **Legacy migration friction**: `.cursorrules` still widely used; migration to `.cursor/rules/` is gradual.
- **No memory approval/review UI**: Unlike Augment Code, there's no built-in way to review or curate memories.
- **Token cost of "Always" rules**: Easy to accidentally bloat context with always-on rules.

## Sources

- [Cursor Rules Docs](https://docs.cursor.com/context/rules)
- [Deep Dive into Cursor Rules (Forum)](https://forum.cursor.com/t/a-deep-dive-into-cursor-rules-0-45/60721)
- [Memory Bank Feature for Cursor (Forum)](https://forum.cursor.com/t/memory-bank-feature-for-your-cursor/71979)
- [Cursor Memory Bank (GitHub)](https://github.com/vanzan01/cursor-memory-bank)
- [Supercharge AI Coding with Cursor Rules and Memory Banks (Lullabot)](https://www.lullabot.com/articles/supercharge-your-ai-coding-cursor-rules-and-memory-banks)
