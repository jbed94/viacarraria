---
name: task-planning
description: Skill for analyzing task requests, gathering context, and generating a structured, persistent task plan in docs/todo.prompt.md.
---

# Task Planning Skill (`task-planning`)

This skill defines the protocol for context gathering and creating persistent task plans saved exclusively to `docs/todo.prompt.md` relative to the main repository root directory.

---

## Planning Rules

1. **Deterministic Path**: The task plan MUST ALWAYS be written to `docs/todo.prompt.md` relative to the workspace root. Never create plan files in floating sub-directories or temporary paths.
2. **Context Gathering**: Read relevant `AGENTS.md` sub-repository documentation and inspect target codebase files before writing plan steps.
3. **Clean Start**: If a planned task is something new, ensure that `docs/todo.prompt.md` is either created afresh or cleared of previous unrelated content to avoid confusion and maintain a single source of truth. Use terminal commands like `rm docs/todo.prompt.md` to remove the old file if necessary (do not modify with tokens). Keep the `docs/todo.prompt.md` file if current planning is a continuation of the work, a natural next suggestion. Clean if a user prompt to do so, or when previous task is complete and new task is not related (is not incremental or not directly connected with last phase).

---

## Task Plan Master Template

When generating a plan, use this exact structural template, which includes an embedded Workflow Recap to preserve execution protocols across context resets:

```markdown
# Task Plan: [Feature / Fix Title]

## Workflow Recap & Execution Protocol
- **Step 1: Planning**: Analyze request, gather codebase context, write persistent plan to `docs/todo.prompt.md` relative to main directory.
- **Step 2: Iteration Execution**: Implement typed feature code & tests on first pass. Execute commands via `mamba-run` skill using sub-repo environments (`mamba run -n viacarraria-<subrepo> <command>`). Fail fast if environment draft is misconfigured.
- **Step 3: Progress Tracking & Interruption Handling**: Mark completed steps (`[x]`). If interrupted by major bugs or missing specs, append entries to Dynamic Tasks & Interruption Log.
- **Step 4: Context Release / Refresh**: After each iteration, invoke `context-refresh` skill to persist handoff snapshot in `docs/todo.prompt.md` and free VS Code Copilot context memory via `/clear`.
- **Step 5: Next Iteration / Completion**: Read `docs/todo.prompt.md` to resume next iteration. Upon plan completion, invoke `suggest-next-steps` skill.

## Objective
High-level description of what will be implemented and the expected behavior.

## Target Sub-Repositories
- [ ] `viacarraria-frontend` (Env: `viacarraria-frontend`)
- [ ] `viacarraria-backend` (Env: `viacarraria-backend`)
- [ ] `viacarraria-database` (Env: `viacarraria-database`)
- [ ] `viacarraria-services` (Env: `viacarraria-services`)
- [ ] `viacarraria-infrastructure` (Env: `viacarraria-infrastructure`)

## Implementation Phases

### Phase 1: Preparation & Schemas
- [ ] Step 1.1: ...
- [ ] Step 1.2: ...

### Phase 2: Core Feature Implementation
- [ ] Step 2.1: ...
- [ ] Step 2.2: ...

### Phase 3: Unit & Integration Testing
- [ ] Step 3.1: Execute test suite via `mamba-run` skill (`mamba run -n viacarraria-<subrepo> ...`).
- [ ] Step 3.2: Verify zero regression in adjacent modules.

### Phase 4: Code Quality & One-Pass Formatting
- [ ] Step 4.1: Run `ruff format` or `biome format` via `mamba-run` skill.

## Dynamic Tasks & Interruption Log
*(Reserved for tracking unexpected bugs, missing user specs, or runtime interruptions during iterations)*

## Execution Log & Verification
*(Appended after each iteration)*

## Context Handoff Snapshot
*(Appended during context release)*
```

---

## Skill Action

Write the compiled task plan directly to `docs/todo.prompt.md` using the file creation tool.
