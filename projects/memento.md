# Memento

## GitHub URL

https://github.com/Agent-on-the-Fly/Memento

## Summary

Memento is a memory-based continual learning framework for LLM agents that achieves continuous improvement from experience without updating model weights. It reframes agent learning as online reinforcement learning over a memory-augmented MDP. The paper is titled "Fine-tuning LLM Agents without Fine-tuning LLMs" (also referred to as AgentFly) and is available at [arxiv.org/abs/2508.16153](https://arxiv.org/abs/2508.16153). It achieves top-1 on the GAIA validation leaderboard with 87.88% Pass@3 and 79.40% on the test set.

## Architecture

Memento uses a **two-stage planner-executor loop** with a case-based memory system:

1. **Case Bank**: A persistent store of successful and failed task trajectories. Each case captures the task decomposition, execution steps, and outcomes.
2. **CBR-driven Planner**: Decomposes incoming tasks and retrieves relevant cases from the Case Bank using a neural case-selection policy. The planner uses retrieved cases to guide task decomposition.
3. **Executor**: Runs each subtask as an MCP client, orchestrating tools (web search, document processing, code execution, image/video analysis) and writing outcomes back to the Case Bank.
4. **Memory-Augmented MDP**: The entire learning process is formulated as an MDP where the state includes both the current task and retrieved memory. A neural policy selects which cases to retrieve and how to apply them.

The system supports built-in MCP tool access for web search, document processing, code execution, and multimedia analysis.

## How It Works

1. A new task arrives and the Planner queries the Case Bank for similar past experiences.
2. The neural case-selection policy ranks and retrieves the most relevant cases.
3. The Planner decomposes the task into subtasks, informed by retrieved case trajectories.
4. The Executor processes each subtask using available tools via MCP.
5. Results (both successes and failures) are logged back into the Case Bank.
6. Over time, the Case Bank accumulates experience, and the case-selection policy improves through online RL -- all without changing the underlying LLM weights.

This creates a feedback loop where the agent gets better at tasks it has seen before and can transfer knowledge to novel out-of-distribution tasks (4.7% to 9.6% absolute improvement on OOD tasks).

## Strengths

- **No fine-tuning required**: Improves agent performance purely through memory, making it model-agnostic and avoiding catastrophic forgetting.
- **Strong benchmark results**: Top-1 on GAIA (87.88%), competitive on DeepResearcher (66.6% F1, 80.4% PM), SimpleQA, and HLE.
- **Transfer learning**: Memory-based approach enables knowledge transfer to out-of-distribution tasks.
- **Scalable**: Case Bank grows over time without degrading base model performance.
- **MCP integration**: Unified tool interface via MCP for extensibility.
- **Backed by research paper**: Rigorous evaluation and clear methodology.

## Weaknesses

- **Case Bank growth**: Unbounded growth of the Case Bank could lead to retrieval latency and storage issues over time without pruning or compaction strategies.
- **Cold start**: With an empty Case Bank, the system has no advantage over a vanilla agent.
- **Case selection quality**: Performance depends heavily on the neural case-selection policy; poor retrieval degrades results.
- **Complexity**: The two-stage planner-executor loop with MDP formulation adds architectural complexity compared to simpler memory approaches (e.g., vector store + RAG).
- **Research-oriented**: Primarily a research system targeting benchmarks rather than a production-ready tool for everyday agent use.
