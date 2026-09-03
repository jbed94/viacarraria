---
name: environment-verification
description: Skill for verifying environment assumptions, detecting missing dependencies, and enforcing immediate fail-fast termination on misconfiguration.
---

# Environment Verification Skill (`environment-verification`)

This skill defines the fail-fast policy for development environment assumptions.

---

## Environment Policy

1. **Pre-Configured Assumption**: The development agent operates under the strict assumption that the environment (compilers, runtimes, Mamba env, Docker containers, node_modules) is pre-configured by the user.
2. **NO Self-Healing / Auto-Repair**: NEVER attempt to install missing platform tools, modify system environment packages, or repair broken Docker / database infrastructure on your own unless explicitly tasked to do so by the user prompt.

---

## Action Protocol

If a command fails due to a missing binary (e.g., `command not found: mamba`), a missing service connection (e.g., PostgreSQL connection refused), or a missing dependency:

1. **Log Environment Failure**: Extract the exact error message and missing requirement.
2. **Fail Fast & Break Conversation**: Report the exact missing dependency or environment configuration issue to the user and IMMEDIATELY halt the conversation trajectory.
3. Do NOT waste tokens trying alternative install scripts or environment workarounds.
