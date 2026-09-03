---
name: context-refresh
description: Skill for context window memory compaction, releasing context in VS Code Copilot, and persisting task state in docs/todo.prompt.md.
---

# Context Refresh Skill (`context-refresh`)

This skill defines the protocol for releasing LLM context memory after an execution iteration while preserving 100% of the active task state in `docs/todo.prompt.md`.

---

## Context Release Protocol for VS Code Copilot

In VS Code Copilot, conversation threads accumulate tokens over long iteration cycles, which can degrade model responsiveness and allocate unnecessary context window capacity. Because each iteration should be as context-independent as possible, context release is executed via persistent file state anchoring:

### 1. Memory Audit & Persistence Check
Before releasing context, verify that `docs/todo.prompt.md` relative to the workspace root contains:
- Checked boxes (`- [x]`) for all finished tasks in the completed iteration.
- Documented file paths created or modified during the iteration.
- Logged test execution commands and empirical outcomes under `## Execution Log & Verification`.

### 2. Append Persistent Context Handoff Snapshot
Append a `## Context Handoff Snapshot` section to the bottom of `docs/todo.prompt.md`:

```markdown
## Context Handoff Snapshot

- **Last Completed Iteration**: Iteration [N]
- **Current Active State**: Ready for Iteration [N+1]
- **Next Target Task**: Phase X Step Y - [Task Title]
- **Modified File References**: `path/to/file1.ts`, `path/to/file2.py`
- **Context Status**: State fully persisted in `docs/todo.prompt.md`. Safe to clear conversation context.
```

### 3. VS Code Copilot Context Clearing Instruction
To free token capacity in VS Code Copilot chat:
1. Provide a concise response to the user summarizing the iteration milestone.
2. Instruct the user or system to issue the `/clear` command in VS Code Copilot chat (or start a new chat turn).
3. Specify that in the fresh conversation turn, mentioning `@docs/todo.prompt.md` or starting with *"Resume task from docs/todo.prompt.md"* will cause the agent to read `docs/todo.prompt.md`, inspect the `Context Handoff Snapshot`, and resume execution seamlessly without carrying past conversation tokens.
