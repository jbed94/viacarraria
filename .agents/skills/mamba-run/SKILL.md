---
name: mamba-run
description: Skill for executing any sub-repository command (testing, building, linting, formatting, database migrations) wrapped in its specific sub-repository mamba environment.
---

# Mamba Execution Skill (`mamba-run`)

This skill defines the mandatory protocol for executing commands across sub-repositories in the Via Carraria monorepo workspace.

## Mamba Environment Isolation Rule

Each sub-repository operates within its own dedicated Mamba environment named after the sub-repository (`viacarraria-<subrepo>`). The root workspace directory does NOT have a Mamba environment and will never require direct command execution from the root level.

```bash
mamba run -n <subrepo_environment_name> <command>
```

---

## Command Patterns per Sub-Repository Mamba Environment

### 1. Frontend Sub-Repository (`viacarraria-frontend`)
- **Mamba Environment**: `viacarraria-frontend`
```bash
# Run unit & component tests
mamba run -n viacarraria-frontend pnpm --filter web test

# Run Vitest in watch mode
mamba run -n viacarraria-frontend pnpm --filter web test:watch

# Run linter & type-checking
mamba run -n viacarraria-frontend pnpm --filter web lint
```

### 2. Backend Sub-Repository (`viacarraria-backend`)
- **Mamba Environment**: `viacarraria-backend`
```bash
# Run backend API unit tests
mamba run -n viacarraria-backend pnpm --filter api test

# Run NestJS integration tests
mamba run -n viacarraria-backend pnpm --filter api test:e2e

# Run linter
mamba run -n viacarraria-backend pnpm --filter api lint
```

### 3. Database Sub-Repository (`viacarraria-database`)
- **Mamba Environment**: `viacarraria-database`
```bash
# Apply Prisma ORM migrations
mamba run -n viacarraria-database npx prisma migrate dev

# Generate Prisma Client types
mamba run -n viacarraria-database npx prisma generate

# Seed pre-baked templates
mamba run -n viacarraria-database npx prisma db seed
```

### 4. Microservices Sub-Repository (`viacarraria-services`)
- **Mamba Environment**: `viacarraria-services`
```bash
# Run Python FastStream worker unit tests
mamba run -n viacarraria-services pytest -f -v parsing-worker/tests/

# Format Python code with Ruff
mamba run -n viacarraria-services ruff format .

# Lint Python code with Ruff
mamba run -n viacarraria-services ruff check --fix .
```

### 5. Infrastructure Sub-Repository (`viacarraria-infrastructure`)
- **Mamba Environment**: `viacarraria-infrastructure`
```bash
# Test Helm templates dry-run
mamba run -n viacarraria-infrastructure helm template ./helm/graph-app

# Run pre-commit quality hooks
mamba run -n viacarraria-infrastructure pre-commit run --all-files
```
