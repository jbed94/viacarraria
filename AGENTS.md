# Via Carraria — Root System Specification

Interactive visual knowledge platform. Converts subjects into Left-to-Right (LR) Directed Acyclic Graphs (DAGs) powered by Spatial GraphRAG search.

---

## System Overview & Implemented Capabilities

### 1. Interactive Knowledge Canvas
- Visualizes subject trees as LR DAGs using React Flow and Dagre.
- Nodes represent skills, courses, concepts. Edges represent prerequisites.
- Node payload: markdown syllabus, study notes, attached source files (PDF, MD, TXT).
- In-place expansion: opens fixed-size scrollable card directly on canvas. Maximized modal view available. Multiple nodes expand simultaneously.

### 2. Spatial GraphRAG Search Engine
- Canvas-native visual retrieval.
- Hybrid vector search (Dense Vectors + BM25 keyword weighting) against Weaviate vector collections filtered by selected node subgraph.
- Neural reranking via Text Embeddings Inference (TEI) reranker model.
- Visual feedback on canvas: matching nodes pulse with badges (+1, +3), unmatched nodes dim, connecting edges animate, camera runs auto-focus (`fitBounds`).
- Document Viewer: highlights parent context span in yellow and exact hit snippets in orange with auto-scroll.
- Extended search: retrieves 1-hop adjacent-node context snippets for registered tiers.

### 3. Graph Builder & Debounced Persistence
- Interactive visual authoring for Pro subscribers. Official system templates owned by system user `"jbed94"`.
- Features: drag handles for edge creation, context toolbars (`+ Add Child Node`), Dagre layout recovery.
- Immediate source upload upon file selection to S3-compatible storage.
- Debounced auto-save (1000ms idle delay) synchronizes canvas nodes and edges to PostgreSQL JSONB columns.

### 4. Plans, Credits & Monetization
- Dynamic plan definitions with configurable feature entitlements, daily query caps, and node limits.
- Credit-based usage system: users track credit balances, consume credits for heavy actions, and execute credit top-ups.
- Stripe / LemonSqueezy webhook integration manages subscription lifecycle.
- Ad monetization: GDPR cookie consent banner, Google AdSense responsive units for Free/Guest tiers, raw telemetry capture (impressions, clicks, dismissals), automated daily aggregation sweeps, safe retention pruning (>90 days).

### 5. Decoupled Administration Panel
- Standalone admin UI running on dedicated port connecting to backend administrative API.
- Modules: system overview, user and subscription management, raw JSONB graph inspection, dynamic plan definitions, ad telemetry rollups and retention sweeps, live audit logging, S3/MinIO cold storage archives with in-browser inspection.

---

## Monorepo Architecture

```
viacarraria/
├── viacarraria-frontend/       # React + React Flow + Zustand + Framer Motion Canvas UI & Decoupled Admin Panel
├── viacarraria-backend/        # Node.js (NestJS) API + CASL + Redis + Weaviate + MinIO/S3 + Admin API
├── viacarraria-database/       # PostgreSQL + Prisma ORM + Migrations & Seeding
├── viacarraria-services/       # Python Ingestion Worker (Docling) + TEI Embeddings & Reranker
└── viacarraria-infrastructure/ # Docker Compose + Helm Charts + KEDA / Karpenter Autoscaling
```

### Submodule Specification Links

- [docs/application.prompt.md](file:///home/jbed94/Projects/viacarraria/docs/application.prompt.md) — Application design specification, UX state machine, PLG rules, agent directives.
- [viacarraria-backend/AGENTS.md](file:///home/jbed94/Projects/viacarraria/viacarraria-backend/AGENTS.md) — Backend API modules, auth, search, plans, billing, admin.
- [viacarraria-database/AGENTS.md](file:///home/jbed94/Projects/viacarraria/viacarraria-database/AGENTS.md) — Database architecture, Prisma models, data integrity.
- [viacarraria-frontend/AGENTS.md](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/AGENTS.md) — Frontend canvas, state machine, modals, overlays, admin panel.
- [viacarraria-infrastructure/AGENTS.md](file:///home/jbed94/Projects/viacarraria/viacarraria-infrastructure/AGENTS.md) — Docker Compose services, Helm manifests, KEDA/Karpenter autoscaling.
- [viacarraria-services/AGENTS.md](file:///home/jbed94/Projects/viacarraria/viacarraria-services/AGENTS.md) — Document parsing, layout extraction, TEI embeddings, reranker, Weaviate storage.

---

## Environment & Packaging Rules

- **Minimal Mamba Environments**: Each submodule contains `environment.yml` with minimal platform tools only (Python/Poetry or Node/pnpm).
- **Native Dependency Managers**:
  - Python: `mamba run -n viacarraria-<subrepo> poetry install`
  - Node: `mamba run -n viacarraria-<subrepo> pnpm install`
- **Standardized Platforms Layout**: Dockerfiles reside under `platforms/docker/`. Orchestration platforms (`platforms/docker-compose/`, `platforms/helm/`) reside exclusively in `viacarraria-infrastructure`.
- **Python Submodule Layout**: Wrapped with Poetry (`pyproject.toml`); source code resides in package-named directory (e.g. `viacarraria_services/`).
