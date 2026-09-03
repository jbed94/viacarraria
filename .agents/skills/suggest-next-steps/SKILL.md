---
name: suggest-next-steps
description: Skill for analyzing completed task outcomes and proposing logical follow-up features, optimizations, or architectural improvements after a completed plan.
---

# Suggest Next Steps Skill (`suggest-next-steps`)

This skill defines how to conclude a fully finished task plan by presenting a structured completion summary and recommending natural follow-up steps.

---

## Action Protocol

When all tasks in `docs/todo.prompt.md` relative to the workspace root have been completed (`[x]`):

1. **Verify Completion**: Confirm that all planned implementation steps and test verifications pass cleanly via sub-repository Mamba environments (`mamba run -n viacarraria-<subrepo> ...`).
2. **Format Report**: Present the completion report using the master Task Completion Summary & Follow-Up Blueprint template below.

---

## Task Completion Summary & Follow-Up Blueprint Template

```markdown
# Task Completion Summary & Follow-Up Blueprint

## Completed Work Summary
- **Primary Task**: [Title of completed task plan]
- **Target Sub-Repositories**: `viacarraria-frontend`, `viacarraria-backend`, etc.
- **Key Milestones Delivered**:
  1. [Milestone 1 detail...]
  2. [Milestone 2 detail...]
- **Empirical Verification**: All unit and integration test suites executed cleanly via `mamba-run`. Zero regression detected.

---

## Suggested Next Steps & Follow-Up Roadmap

### 1. Immediate Follow-Up: [Feature / Test Title]
- **Category**: Integration / Testing / Feature Extension
- **Description**: Concise overview of the next logical task to undertake.
- **Rationale**: Why this step directly builds upon the completed milestones.
- **Target Sub-Repository**: `viacarraria-<subrepo>` (Env: `viacarraria-<subrepo>`)

### 2. Architectural Optimization: [Optimization Title]
- **Category**: Performance / Caching / Refactoring
- **Description**: Proposed optimization or technical debt reduction step.
- **Rationale**: Improves execution speed, memory footprint, or code maintainability.
- **Target Sub-Repository**: `viacarraria-<subrepo>` (Env: `viacarraria-<subrepo>`)

### 3. Future Scope Expansion: [Expansion Title]
- **Category**: UX Enhancement / Product-Led Growth
- **Description**: Broad feature enhancement for future iteration cycles.
- **Rationale**: Aligns with long-term platform capabilities and user entitlements.
- **Target Sub-Repository**: `viacarraria-<subrepo>` (Env: `viacarraria-<subrepo>`)
```
