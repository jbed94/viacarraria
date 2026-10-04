---
name: cheap-calling
description: >
  Guidelines for enforcing strict token discipline in terminal executions, 
  file inspections, and file modifications. Use always when bash commands 
  are involved, in connection with `mamba-run` skill and when modifying files.
  Load when exact `cheap-calling` skill is needed.
---

## Cheap Calling & Token Conservation Rules

The skill defines rules and best practices to decrease the consumption of tokens. 

### 1. Terminal Command Execution

First of all, always wrap standard shell commands with `rtk` to compress output:
```bash
rtk <command> [args...]
# Examples:
rtk git status -s
rtk cargo test
rtk npm run build
```

When working with `mamba`, `conda`, or `poetry` environment wrappers, never wrap the launcher binary with `rtk`. Inject `rtk` immediately before the target payload command:

```bash
# Correct:
mamba run -n <env> rtk python script.py
conda run -n <env> rtk pytest tests/
poetry run rtk flask run

# Forbidden:
rtk mamba run -n <env> python script.py
rtk conda run -n <env> pytest
```

`rtk` commands relevant to this workspace:
```bash
  ls             List directory contents with token-optimized output
  tree           Directory tree with token-optimized output
  read           Read file with intelligent filtering
  find           Find files with compact tree output (accepts native find flags)
  grep           Compact grep - strips whitespace, truncates, groups by file
  diff           Ultra-condensed diff (only changed lines)
  git            Git commands with compact output
  pnpm           pnpm commands with ultra-compact output
  vitest         Vitest commands with compact output
  jest           Jest commands with compact output
  tsc            TypeScript compiler with grouped error output
  prisma         Prisma commands with compact output (no ASCII art)
  pytest         Pytest test runner with compact output
  ruff           Ruff linter/formatter with compact output
  docker         Docker commands with compact output
  kubectl        Kubectl commands with compact output
  test           Run tests and show only failures
  err            Run command and show only errors/warnings
  smart          Generate 2-line technical summary (heuristic-based)
```
  
### 2. File Editing & Modification

* **Never regenerate whole files:** Modifying an existing file must use surgical block replacements (`replace_file_content`, `multi_replace_file_content`), localized `patch`/`diff`, or `sd`.
* **Zero cosmetic rewrites:** Restrict modifications strictly to target lines and their immediate callers. Do not reformat untouched code blocks.
* **No manual linting and formatting:** Do not manually run linters or formatters on files. Any necessary linting or formatting should be handled automatically by the appropriate tools or scripts.
