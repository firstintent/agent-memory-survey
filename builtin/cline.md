# Cline Memory Bank System

## Overview

Cline's Memory Bank is a documentation-driven methodology that uses structured markdown files and Mermaid diagrams to give the AI persistent project context. It is implemented through Cline's custom instructions feature, not as a hardcoded feature -- making it a "methodology" rather than a traditional built-in memory system. This distinction is important: Memory Bank is a pattern that users opt into via custom instructions.

## Storage Locations

| Component | Path | Purpose |
|-----------|------|---------|
| Custom instructions | Cline settings (global) | Memory Bank methodology rules |
| Project brief | `memory-bank/projectbrief.md` | Foundation document, core requirements |
| Active context | `memory-bank/activeContext.md` | Current work focus, recent changes |
| System patterns | `memory-bank/systemPatterns.md` | Architecture, design patterns |
| Tech context | `memory-bank/techContext.md` | Stack, dependencies, tooling |
| Progress | `memory-bank/progress.md` | What works, what's left, blockers |
| Product context | `memory-bank/productContext.md` | (Optional) Product goals, user needs |
| Cline rules | `.clinerules` (project root) | Project-specific rules |

## Memory Types

### Core Memory Bank Files

The Memory Bank consists of required core files, all in markdown format:

1. **projectbrief.md**: Foundation document that shapes all other files. Created at project start. Defines core requirements, goals, and scope. Source of truth for the project.

2. **activeContext.md**: Current work focus, recent changes, next steps, active decisions and considerations. Updated frequently -- the "working memory" of the system.

3. **systemPatterns.md**: System architecture, key technical decisions, design patterns in use, component relationships. Includes Mermaid diagrams for visual architecture representation.

4. **techContext.md**: Technology stack, development environment, dependencies, technical constraints.

5. **progress.md**: What's working, what's not, known issues, feature completion status.

### Mermaid Diagrams

A distinctive aspect of Cline's Memory Bank. Mermaid diagrams are embedded in the custom instructions themselves (not just in the memory files) to describe:

- The Memory Bank update workflow
- File dependency relationships
- Decision trees for when to update which files
- Architecture visualizations in systemPatterns.md

Mermaid is used because it provides unambiguous, formally structured workflow descriptions that are easier for AI to parse than natural language paragraphs. Each node, connection, and decision point is explicitly defined.

### .clinerules (Project Rules)

A file in the project root that provides project-specific instructions, similar to `.cursorrules` or `CLAUDE.md`. Loaded automatically for every interaction in that project.

## Persistence Scope

- **Memory Bank files**: Persist in the repo. Version-controllable via git. Shared with team.
- **Custom instructions**: Persist in Cline settings. Local to the user's installation.
- **.clinerules**: Persists in the repo. Shared via git.
- **Session context**: Not persisted. Each new task starts by reading the Memory Bank files.

## The Memory Bank Workflow

The methodology operates on a continuous cycle:

1. **Read**: At the start of every task, Cline reads all Memory Bank files to rebuild context.
2. **Verify**: Confirm understanding of current state before making changes.
3. **Execute**: Perform the requested work.
4. **Update**: Update relevant Memory Bank files to reflect changes made.

This read-verify-execute-update cycle ensures the Memory Bank stays current. The custom instructions explicitly direct Cline to update `activeContext.md` and `progress.md` after every significant action.

## User Control

- All Memory Bank files are plain markdown, directly editable by users.
- The methodology is opt-in via custom instructions. Users can modify the workflow.
- `projectbrief.md` is the only file users typically write manually; others are maintained by the agent.
- Users can create variations of the Memory Bank structure for different project types.
- Community-shared custom instruction templates available for different workflows.

## Implementation Details

- Memory Bank is implemented entirely through Cline's custom instructions feature -- no special code in Cline itself.
- The instructions use Mermaid flowcharts to define the update workflow unambiguously.
- Every task begins with a full read of all Memory Bank files, consuming context tokens.
- No selective loading or lazy loading -- all files are read every time.
- No vector database or semantic search. Pure file-based retrieval.
- Community variations exist: some add MCP-based memory servers, phase-based workflows, or additional specialized files.

## Strengths

- **Transparent and auditable**: Every piece of "memory" is a plain markdown file in the repo.
- **Version-controllable**: Memory Bank lives in the repo. Full git history of how project context evolved.
- **Team-shareable**: Memory Bank files committed to git give new team members (and new Cline sessions) full context.
- **Mermaid diagrams are clever**: Using a formal diagramming language for workflow definitions reduces ambiguity in instructions.
- **Methodology, not feature**: Can be adapted, extended, and customized. Not locked into a specific implementation.
- **Self-updating**: The agent maintains its own memory files as part of the workflow.

## Weaknesses

- **Token-heavy**: Reading all Memory Bank files at the start of every task consumes significant context. No selective loading.
- **Manual setup required**: Users must configure custom instructions and create initial files. Not zero-config.
- **No semantic search**: All files loaded verbatim. As projects grow, Memory Bank files may become too large for the context window.
- **Methodology fragility**: Because it's implemented via custom instructions, the agent may not always follow the update cycle perfectly. Drift happens.
- **No built-in support**: Cline itself has no Memory Bank awareness. It's a community pattern, not a first-class feature.
- **Duplication with .clinerules**: The relationship between .clinerules and Memory Bank custom instructions can be confusing.
- **Scaling challenges**: For large projects, maintaining accurate Memory Bank files becomes a significant overhead.

## Sources

- [Memory Bank: How to Make Cline an AI Agent That Never Forgets (Cline Blog)](https://cline.bot/blog/memory-bank-how-to-make-cline-an-ai-agent-that-never-forgets)
- [Memory Bank Docs (Cline)](https://docs.cline.bot/features/memory-bank)
- [Memory Bank Custom Instructions (GitHub)](https://github.com/nickbaumann98/cline_docs/blob/main/prompting/custom%20instructions%20library/cline-memory-bank.md)
- [Cline Memory Bank Variations (Threads)](https://www.threads.com/@cline.bot/post/DJHq0ODRlH-)
