---
name: progress-tracking
description: >-
  Skill for updating task progress, checking completed steps, 
  handling interruptions. Related to docs/todo.prompt.md. 
  This skill is always used after planning and execution iterations.
  Load when exact `progress-tracking` skill is needed.
---

# Progress Tracking Skill (`progress-tracking`)

This skill defines how to maintain real-time task accuracy in `docs/todo.prompt.md` after each execution iteration. Never skip updating the progress log, to avoid context degradation.

## Progress Update Protocol

After an execution iteration completes (or gets interrupted), update `docs/todo.prompt.md` using the following rules:

### 1. Update Task Statuses
- Change completed step checkboxes from `- [ ]` to `- [x]`.
- An iteration may complete multiple steps across phases. Mark all finished steps.
- Never check again functionality.
- Never inspect functionality to check them, they should be checked immediately after execution iteration.

### 2. Handle Interruption Events

Classify the interruption and respond accordingly:

**Fixable interruption** (bug, missing edge case, small dependency issue):
- Append an entry under `## Dynamic Tasks & Interruption Log` in `docs/todo.prompt.md`.
- Document the exact blocking issue, reproduction context, and pending decision.
- Add new sub-task checkboxes to resolve the interruption once unblocked.

**Major refactor required** (fundamentally wrong approach, architectural mismatch, cascading failures after 2+ fix attempts):
- Do NOT attempt the refactor within the current task scope.
- Halt execution immediately.
- Invoke `suggest-next-steps` skill with the refactor framed as the primary follow-up (using `hypothesis`-based format if the task was driven by `agent-problem-solving`).

### 3. Log Empirical Test Verification
Append an entry under `## Execution Log & Verification`:

```markdown
### Iteration [N] Snapshot - [Timestamp]
- **Completed Steps**: Phase 2 Step 2.1, Step 2.2
- **Command Executed**: `pnpm --filter api test` (via `mamba-run`)
- **Outcome**: 15 tests passed, 0 failed.
- **Interruption Status**: None / [Description if interrupted].
```

Keep only the **last 3 iteration snapshots** in the Execution Log. When appending a 4th, remove the oldest. The log exists for continuity across iterations, not as history.

---

## Skill Action

Apply updates to `docs/todo.prompt.md` relative to the workspace root using `replace_file_content` or `write_to_file`.
