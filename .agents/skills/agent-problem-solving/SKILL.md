---
name: agent-problem-solving
description: >-
  Incremental problem solving via means-ends decomposition and hypothesis-driven
  refinement. Use when designing, building, or improving a system or functionality;
  when a clear problem is stated and a solution must be constructed step by step;
  or when evaluating and upgrading an existing approach. Produces a minimal viable
  starting point and iteratively improves it through targeted hypotheses on each
  component until goal and side criteria are satisfied.
  Load when exact `agent-problem-solving` skill is needed.
---

# Incremental Problem Solving

Solve problems by constructing a minimal starting solution and refining it step by
step through hypothesis-driven improvement. Applies to software development, system
design, and general analytical problem solving.

Uses the [hypothesis](../hypothesis/SKILL.md)
skill for individual improvement steps.

## Cognitive Foundations

| Source | Key insight |
|---|---|
| **Newell & Simon** — *Human Problem Solving* (1972) | Problem solving = **search through a problem space**. Means-ends analysis identifies the gap between current and goal state, then selects operators to reduce it. Subgoaling decomposes large gaps into manageable steps. |
| **Simon** — *Bounded Rationality & Satisficing* (1956) | Under real constraints (time, information, compute), start with a **"good enough" solution** that meets minimum criteria, then raise aspiration levels iteratively. |
| **Pólya** — *How to Solve It* (1945) | Understand → Plan → Execute → **Reflect**. Solve a simpler version first. Use analogy, work backward from the goal. |
| **Vygotsky** — *Zone of Proximal Development* (1978) | Progress happens at the boundary of current capability. **Scaffold** complexity: each step should be a manageable stretch from the previous state, not a leap. |
| **Popper** — *Falsificationism* (1959) | Each refinement step is a hypothesis. It must be testable and rejectable. |

## Core Principle

> Start from anything that structurally addresses the problem — the simplest
> mechanism, topology, or idea that could work. Then improve it one component
> at a time, where each improvement is a hypothesis with goal and side criteria,
> verified before moving on.

This is means-ends analysis applied recursively: identify the largest gap between
the current state and the goal, formulate a hypothesis for closing it, test, decide,
then re-evaluate remaining gaps.

## Process

### 1. Define the Problem Space

- **Goal state:** What does "solved" look like? Define observable acceptance criteria.
- **Current state:** What exists now? (Nothing, a broken system, a working-but-insufficient system.)
- **Constraints:** Time, tech stack, compatibility, performance budget, team conventions.
- **Decomposition:** Break the problem into **components** (functional units, layers, concerns). Each component can be addressed independently.

### 2. Construct the Minimal Starting Solution

Build (or identify) the simplest thing that **structurally addresses every component**, even if each one is a stub, a naive algorithm, or a hardcoded value.

Criteria for the starting solution:
- Addresses the problem's core topology/mechanism.
- Compiles, runs, or can be reasoned about concretely.
- Is deliberately incomplete — it exists to be improved.

This is satisficing (Simon): set aspiration levels low enough to produce a concrete
starting point, then raise them.

### 3. Identify Gaps (Means-Ends Analysis)

Compare the current solution to the goal state. For each component, list what is
missing, wrong, or insufficient. Rank gaps by impact — start with the gap whose
resolution changes the most.

### 4. Improve One Component (Hypothesis Cycle)

For the highest-impact gap, run one cycle of the
[hypothesis skill](../hypothesis/SKILL.md):

1. **Formulate** a hypothesis for closing the gap (if → then, goal + side criteria).
2. **Design** the cheapest falsification test.
3. **Execute** and observe.
4. **Decide:** accept, reject, or refine.
5. **Integrate** the accepted improvement into the solution.

After integration, the solution state has changed — return to step 3 to re-evaluate
remaining gaps. Priorities may shift after each improvement.

### 5. Terminate

Stop when:
- All acceptance criteria from step 1 are met.
- Remaining gaps are below the threshold of diminishing returns.
- Constraints (time, scope) are reached and the current solution satisfices.

### 6. Reflect

- Does the solution generalize or is it tightly coupled to the specific problem?
- Which hypotheses were rejected and why? (Capture for future reuse.)
- Was complexity scaffolded well, or did a step try to change too much at once?

## Track A: Software Development

**Scenario:** Design or improve a feature, module, or system.

| Step | Concretely |
|---|---|
| Define problem space | Requirements, API contract, affected modules, constraints (stack, perf, security). |
| Minimal starting solution | Simplest implementation that type-checks and handles the happy path. Hardcoded values, in-memory state, no edge cases. |
| Identify gaps | Missing error handling, no persistence, poor performance, missing auth, no tests. |
| Improve one component | Pick the most impactful gap. Formulate hypothesis (e.g., "If we replace the O(n²) scan with an index lookup, response time drops below 50ms"). Test. Integrate. |
| Terminate | Feature meets acceptance criteria and passes CI. |
| Reflect | Identify reusable patterns; note rejected approaches for ADR. |

**Anti-pattern:** Building the "final" architecture upfront before verifying that the core mechanism works. Always prove the mechanism first, then layer on complexity.

## Track B: General Problem Solving

**Scenario:** Analyze, design, or evaluate any system, approach, or decision.

| Step | Concretely |
|---|---|
| Define problem space | Observable symptoms, desired outcomes, boundaries, known evidence. |
| Minimal starting solution | The simplest model, explanation, or design that accounts for the core structure of the problem. May be a sketch, a diagram, a formula, a prototype. |
| Identify gaps | Where does the model fail to predict, explain, or satisfy constraints? |
| Improve one component | Target one failing component. Hypothesize a mechanism, test it (literature, calculation, simulation, prototype), decide. |
| Terminate | Model/design accounts for all critical phenomena within acceptable accuracy. |
| Reflect | What generalizes? What was specific to this instance? |

**Anti-pattern:** Attempting to build the complete model in one shot. Complex systems are understood one interaction at a time.

## Anti-Patterns

| Anti-pattern | Root cause | Remedy |
|---|---|---|
| Building the full solution before testing the core idea | Premature optimization / planning fallacy | Prove the mechanism works with the minimal version first |
| Improving multiple components at once | Confounds cause and effect | One hypothesis per iteration; isolate variables |
| Skipping gap re-evaluation after each step | Anchoring on initial priority list | Re-rank after every integration; priorities shift |
| Refusing to satisfice | Perfectionism / loss aversion | Set explicit "good enough" thresholds upfront |
| No reflection between iterations | Action bias | Pause after each cycle; check for bias and generalization |

## Template

```markdown
## Problem: [Title]

### Problem Space
- **Goal state:** [Observable acceptance criteria]
- **Current state:** [What exists now]
- **Constraints:** [Time, stack, performance, compatibility]
- **Components:** [Decomposed functional units]

### Minimal Starting Solution
[Description of the simplest thing that structurally addresses the problem]

### Iteration [N]
- **Gap:** [Largest difference between current state and goal]
- **Hypothesis:** If [change], then [testable prediction]
  - Goal criteria: [What must hold]
  - Side criteria: [Constraints, trade-offs]
- **Test:** [Cheapest falsification]
- **Result:** [Observed outcome]
- **Decision:** [Accept / Reject / Refine]
- **New state:** [Updated solution description after integration]

### Termination
- **Criteria met:** [Which acceptance criteria are satisfied]
- **Residual gaps:** [Gaps below threshold or deferred]

### Reflection
- [Key learnings, rejected approaches, generalizable patterns]
```
