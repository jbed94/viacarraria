---
name: hypothesis
description: >-
  Structured hypothesis formulation and verification. Use when analyzing a problem,
  designing a solution, choosing between approaches, debugging unexpected behavior,
  evaluating a library/tool, or proposing an architectural change. Activates disciplined
  reasoning: state what you believe, why, what would prove you wrong, and test it.
  Load when exact `hypothesis` skill is needed.
---

# Hypothesis Formulation & Verification

A concise manual for disciplined conjecture-and-test reasoning in software engineering,
derived from core cognitive science principles.

## Cognitive Foundations

| Source | Key insight |
|---|---|
| **Popper** — *The Logic of Scientific Discovery* (1959) | A hypothesis has value only if it is **falsifiable**. Seek disconfirmation, not confirmation. |
| **Newell & Simon** — *Human Problem Solving* (1972) | Problem solving = **search through a problem space**. Hypotheses constrain the search; heuristics (means-ends analysis, subgoaling) guide it. |
| **Pólya** — *How to Solve It* (1945) | Four phases: understand → plan → execute → **reflect**. The reflection step is where learning happens. |
| **Chamberlin** (1890) / **Platt** (1964) | Maintain **multiple working hypotheses** simultaneously. Design crucial experiments that eliminate candidates. Prevents attachment to a single ruling theory. |
| **Wason** — *2-4-6 Task* (1960) | Humans default to **confirmation bias**: they test only cases they expect to succeed. Deliberately generate disconfirming tests. |
| **Kahneman & Tversky** — *Prospect Theory / Heuristics & Biases* (1974) | Bounded rationality produces systematic errors (anchoring, availability, framing). Awareness of these biases is the first defense. |

## Process

### 1. Frame the Problem

Before hypothesizing, ensure you understand the problem space:
- **What** is the observable symptom, requirement, or opportunity?
- **Where** are the boundaries? What is in scope, what is not?
- **What do you already know** (evidence, constraints, prior art)?

### 2. Formulate Hypotheses

State each hypothesis in falsifiable **if → then** form:

> **If** [proposed cause / mechanism / design choice], **then** [observable, testable prediction].

For each hypothesis define:
- **Goal criteria** — the desired outcome that must hold for the hypothesis to succeed.
- **Side criteria** — constraints, risks, trade-offs, second-order effects that may disqualify an otherwise valid hypothesis (performance budget, maintenance cost, ecosystem health, security surface, license compatibility).

When the problem admits it, formulate **≥ 2 competing hypotheses** (Chamberlin/Platt). This counteracts confirmation bias and ruling-theory attachment.

### 3. Design Verification

Choose the **cheapest test that can falsify** the hypothesis:
- Prefer a single command, a minimal reproducer, a type-check, or a docs lookup over a full integration test.
- If two hypotheses compete, design a **crucial test** — one whose outcome eliminates at least one candidate.
- Explicitly plan for **disconfirming evidence** (Wason): what result would prove the hypothesis wrong?
- When the hypothesis involves code behavior, prefer writing a **failing test** as the falsification mechanism — it both verifies the hypothesis and becomes part of the test suite.

### 4. Execute & Observe

Run the test. Record the actual outcome without interpreting it through the lens of your expectation.

### 5. Decide

- Evaluate against **both** goal and side criteria. A hypothesis that meets its goal but violates a side constraint is rejected.
- If falsified → update beliefs, refine or replace the hypothesis, return to step 2.
- If corroborated → adopt provisionally. Remember: corroboration ≠ proof (Popper).

### 6. Reflect

Brief retrospective (Pólya's "look back"):
- What did you learn?
- Does the finding generalize?
- Were there biases in your reasoning?

## Anti-Patterns

| Anti-pattern | Cognitive root | Remedy |
|---|---|---|
| Testing only happy-path cases | Confirmation bias (Wason) | Explicitly list what would falsify the hypothesis and test that first |
| Anchoring on the first idea | Anchoring bias (Kahneman) | Force ≥ 2 alternatives before committing |
| Over-engineering the test | Scope creep / planning fallacy | Find the minimum viable falsification |
| Ignoring side criteria in decision | Goal fixation | Evaluate side criteria with equal weight |
| Treating corroboration as proof | Verification bias (Popper) | State residual uncertainty; revisit if new evidence appears |

## Template

```markdown
### Hypothesis [N]

**Problem:** [What are we trying to explain, solve, or decide?]
**If** [condition / mechanism / design choice], **then** [testable prediction].
**Goal criteria:** [What must hold for success]
**Side criteria:** [Constraints, risks, trade-offs — if any]
**Falsification test:** [Cheapest test that could disprove this]
**Competing hypothesis:** [Alternative explanation — if applicable]

**Result:** [Observed outcome]
**Decision:** [Accept / Reject / Refine → next iteration]
**Reflection:** [What was learned; any bias spotted]
```