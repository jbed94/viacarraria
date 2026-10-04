---
name: python-submodule
description: >-
  Architecture and conventions for Python submodules in the Via Carraria monorepo.
  Use when creating, modifying, or reviewing Python packages, Poetry configurations,
  Pydantic models, FastStream workers, or async Python code in any Python sub-repository.
  Load when exact `python-submodule` skill is needed.
---

# Python Submodule Architecture

Conventions and guidelines for Python sub-repositories in Via Carraria.

## Package Structure

Every Python submodule follows this layout:

```
viacarraria-<subrepo>/
├── environment.yml              # Mamba env: python + poetry only
├── pyproject.toml               # Poetry package definition
├── viacarraria_<subrepo>/       # Source package (underscore naming)
│   ├── __init__.py
│   ├── <module>/                # Feature modules
│   │   ├── __init__.py
│   │   ├── models.py            # Pydantic v2 models
│   │   └── ...
│   └── scripts/                 # Utility scripts
├── tests/                       # Pytest test suite
│   ├── test_*.py
│   └── conftest.py
└── platforms/
    └── docker/                  # Dockerfiles per service
```

## Rules

1. **Poetry wrapping required.** Every Python submodule uses `pyproject.toml` with Poetry as build backend.
2. **Source folder = package name.** Source code resides in `viacarraria_<subrepo>/` (underscore, matching the Poetry package `packages` directive).
3. **Mamba env is minimal.** `environment.yml` contains only `python` and `poetry`. All application dependencies are managed by Poetry.
4. **Pydantic v2 for data schemas.** All data structures, config models, and API contracts use Pydantic `BaseModel` with strict typing.
5. **Async-first.** Workers and I/O-bound code use `asyncio`. Sync wrappers only where external libraries demand it.
6. **Ruff for formatting and linting.** Single tool for both. Config in `pyproject.toml` under `[tool.ruff]`.
7. **Pytest with asyncio_mode = "auto".** Tests in `tests/` directory. Async test functions run automatically.

## Poetry Configuration Pattern

```toml
[tool.poetry]
name = "viacarraria-<subrepo>"
packages = [{ include = "viacarraria_<subrepo>" }]

[tool.poetry.dependencies]
python = "^3.12"

[tool.poetry.group.dev.dependencies]
pytest = "^8"
pytest-asyncio = "^1"
ruff = "^0.12"

[tool.pytest.ini_options]
testpaths = ["tests"]
asyncio_mode = "auto"

[tool.ruff]
target-version = "py312"
line-length = 100

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B", "ASYNC"]

[build-system]
requires = ["poetry-core>=2.0.0,<3.0.0"]
build-backend = "poetry.core.masonry.api"
```

## FastStream Worker Pattern

Workers subscribe to RabbitMQ queues via FastStream's `RabbitBroker`:

```python
from faststream.rabbit import RabbitBroker, RabbitQueue

broker = RabbitBroker(url="amqp://...")
queue = RabbitQueue("document_parsing_queue", durable=True)

@broker.subscriber(queue)
async def handle(msg: JobMessage) -> None:
    ...
```

- Use priority headers for tier-based scheduling.
- Configure Dead Letter Exchange (DLX) + Dead Letter Queue (DLQ) for failure resilience.
- Stream progress to Redis for real-time frontend updates.

## Platforms Layout

- Dockerfiles reside in `platforms/docker/` within the submodule.
- Docker Compose and Helm orchestration reside exclusively in `viacarraria-infrastructure`.
