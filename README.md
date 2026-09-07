# Via Carraria

Via Carraria is a multi-repository workspace for an interactive knowledge canvas
and Spatial GraphRAG platform. This root repository tracks five independently
tooled submodules:

- `viacarraria-frontend`: React, Vite, React Flow, Zustand, and Biome
- `viacarraria-backend`: NestJS API and service integrations
- `viacarraria-database`: Prisma, PostgreSQL migrations, and template seeding
- `viacarraria-services`: Poetry-based parsing and embedding services
- `viacarraria-infrastructure`: Docker Compose, Helm, and platform validation

## Bootstrap

Each submodule owns a minimal Mamba environment in `environment.yml`. Create the
required environment explicitly, then install application dependencies with the
native package manager inside it. No root environment combines submodule tools.

Local service topology is defined in
`viacarraria-infrastructure/platforms/docker-compose`; production packaging is
initialized in `viacarraria-infrastructure/platforms/helm/graph-app`.

The repository currently contains development scaffolding only. Domain models,
API features, graph UX, worker pipelines, and deployable Kubernetes templates
remain intentionally unimplemented.
