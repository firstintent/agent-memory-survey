# Aider Memory System

## Overview

Aider uses a repository map built on tree-sitter parsing with a tag-based cache, combined with conventions files for persistent instructions. The "LanceDB vector database" aspect comes from Aider-Desk (a community GUI wrapper), not core Aider itself. Core Aider's memory is the repo map + conventions files + chat history within a session.

## Storage Locations

| Component | Path | Purpose |
|-----------|------|---------|
| Tag cache | `.aider.tags.cache.v3/` (project root) | Cached tree-sitter parse results |
| Conventions | `.aider/conventions.md` or custom path | Persistent instructions |
| Chat history | `.aider.chat.history.md` | Session transcript log |
| Input history | `.aider.input.history` | Command-line input history |
| Ignore file | `.aiderignore` | Exclude files from repo map |
| Aider-Desk memories | `.aider-desk/memory/` | LanceDB vector store (Aider-Desk only) |

## Memory Types

### Repository Map (Core Memory)

Aider's primary "memory" is its dynamically generated repository map. On each interaction, Aider:

1. Parses all tracked files using tree-sitter to extract tags (function/class/method definitions and references)
2. Builds a graph of cross-file relationships
3. Ranks files by relevance to the current conversation using PageRank-like scoring
4. Generates a condensed map showing the most relevant code structure

The repo map gives the LLM a bird's-eye view of the codebase without loading every file. Only the most relevant portions are included in the prompt.

### Tag Cache

Parse results are cached in `.aider.tags.cache.v3/` using `diskcache.Cache` (SQLite-backed). Each entry stores file modification time and extracted tags. If a file hasn't changed, cached tags are reused. Falls back to in-memory dict if SQLite errors occur.

### Conventions File

A markdown file (default `.aider/conventions.md`, configurable via `--conventions` flag) containing persistent instructions:

- Coding style preferences
- Architecture decisions
- Testing requirements
- Forbidden patterns

Loaded into the system prompt on every interaction. Analogous to `CLAUDE.md` or `.cursorrules`.

### Chat History

`.aider.chat.history.md` logs the full session transcript. Not loaded into future sessions automatically -- it's a reference log, not active memory. Within a session, Aider maintains the full conversation context.

### Aider-Desk Vector Memory (Community Extension)

Aider-Desk (a GUI wrapper) adds LanceDB-based vector memory:

- Uses `Xenova/all-MiniLM-L6-v2` embeddings (384-dimensional)
- Stores memories in `.aider-desk/memory/` per project
- Supports semantic search across memories
- Memory isolation per project prevents cross-contamination

This is **not** part of core Aider.

## Persistence Scope

- **Repo map**: Regenerated every interaction. The tag cache persists across sessions for performance.
- **Conventions**: Persists in the repo. Version-controllable.
- **Chat history**: Persists as a log file but not re-ingested.
- **Session context**: Not persisted. Each new `aider` invocation starts fresh.

## User Control

- `.aiderignore` controls which files are excluded from the repo map (uses `.gitignore` syntax).
- `--map-tokens` controls the token budget for the repo map (default: 1024).
- `--map-refresh` controls how often the map is regenerated (`auto`, `always`, `files`, `manual`).
- Conventions file is directly editable.
- `/map` command displays the current repo map.
- No auto-memory creation -- all persistent instructions are human-authored.

## Implementation Details

- Tree-sitter is used for language-aware parsing (supports Python, JavaScript, TypeScript, Go, Rust, Java, C/C++, and many more).
- The repo map uses a ranking algorithm based on tag references: files that are referenced more by the currently-discussed files rank higher.
- Map generation involves: parse tags -> build reference graph -> rank by relevance -> render top-N files with their key definitions.
- In-memory caches (`map_cache`, `tree_cache`) avoid redundant rendering within a session.
- Universal ctags is a legacy fallback if tree-sitter doesn't support a language.

## Strengths

- **Intelligent code context**: The repo map dynamically selects the most relevant code, not just files. Better than loading everything or nothing.
- **Language-aware**: Tree-sitter parsing understands code structure, not just text patterns.
- **Efficient caching**: SQLite-backed tag cache avoids re-parsing unchanged files.
- **Simple conventions file**: Easy to set up persistent instructions.
- **Token-efficient**: Repo map respects a configurable token budget, keeping prompt size manageable.
- **No cloud dependency**: Everything runs locally.

## Weaknesses

- **No auto-memory**: Aider doesn't learn from sessions. Each invocation starts with only the repo map and conventions.
- **No cross-session continuity**: Chat history is logged but not re-ingested. No summarization or compression.
- **Repo map is ephemeral**: While the tag cache persists, the actual map is regenerated each time. No persistent "learned" code understanding.
- **Conventions file is manual**: Must be authored and maintained by the user.
- **No memory review UI**: No built-in way to browse or manage what Aider "knows."
- **Vector memory requires third-party wrapper**: LanceDB support is only in Aider-Desk, not core Aider.

## Sources

- [Aider Repository Map Docs](https://aider.chat/docs/repomap.html)
- [Repository Mapping System (DeepWiki)](https://deepwiki.com/Aider-AI/aider/4.1-repository-mapping)
- [Aider-Desk Memory System (DeepWiki)](https://deepwiki.com/hotovo/aider-desk/3.7-memory-system)
- [Building a Better Repository Map with Tree-Sitter](https://aider.chat/2023/10/22/repomap.html)
