---
name: progress-tracking
description: Skill for updating task progress, checking completed steps, tracking interruptions, and recording execution notes in docs/todo.prompt.md.
---

# Progress Tracking Skill (`progress-tracking`)

This skill defines how to maintain real-time task accuracy in `docs/todo.prompt.md` after each execution iteration.

---

## Progress Update Protocol

After an execution iteration completes (or gets interrupted), update `docs/todo.prompt.md` using the following rules:

### 1. Update Task Statuses
- Change completed step checkboxes from `- [ ]` to `- [x]`.
- An iteration may complete multiple steps across phases. Mark all finished steps.

### 2. Handle Interruption Events
If an iteration encounters an unexpected situation (e.g., a major blocking bug that cannot be resolved on the fly, or a missing specification requiring user clarification):
- Append an entry under `## Dynamic Tasks & Interruption Log` in `docs/todo.prompt.md`.
- Document the exact blocking issue, reproduction context, and pending decision.
- Add new sub-task checkboxes to resolve the interruption once unblocked.

### 3. Log Empirical Test Verification
Append an entry under `## Execution Log & Verification`:

```markdown
### Iteration [N] Snapshot - [Timestamp]
- **Completed Steps**: Phase 2 Step 2.1, Step 2.2
- **Command Executed**: `pnpm --filter api test` (via `mamba-run`)
- **Outcome**: 15 tests passed, 0 failed.
- **Interruption Status**: None / [Description if interrupted].
```

---

## Skill Action

Apply updates to `docs/todo.prompt.md` relative to the workspace root using `replace_file_content` or `write_to_file`.
