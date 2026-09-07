# Via Carraria — Root System Specification

Via Carraria is an interactive visual knowledge organization, career roadmap, and skill acquisition platform. It transforms complex educational domains (e.g., computer science, medicine, language learning) into interactive Directed Acyclic Graphs (DAGs) powered by **Spatial GraphRAG** (Visual Retrieval-Augmented Generation).

---

## General Application Overview & Core Value Proposition

### What is Via Carraria?
Via Carraria reimagines how people explore, structure, and query complex knowledge domains. Traditional AI study tools present knowledge as linear chat logs or document lists, which obscure the hierarchical dependencies between skills and concepts. Via Carraria solves this by fusing interactive Directed Acyclic Graphs (DAGs) with vector search directly on the canvas space.

### Core Capabilities & User Features
1. **Interactive Knowledge Canvas**:
   - Visualizes subject roadmaps and skill trees as Left-to-Right (LR) DAGs.
   - Nodes represent individual topics, courses, or skills (e.g., *Linear Algebra*, *Database Systems*, *Anatomy*).
   - Edges represent prerequisites and topic dependencies (e.g., *Calculus I* $\to$ *Differential Equations*).
   - Each node encapsulates rich markdown descriptions, syllabi, notes, and attached primary source files (PDF, Markdown, TXT). Clicking a node expands it in-place to a fixed-size scrollable card, or opens a maximized full-screen view.

2. **Spatial GraphRAG Search**:
   - Replaces conventional chat windows with canvas-native visual retrieval.
   - Users focus an AI search field, select target nodes/sources (or keep defaults), and submit natural language questions (e.g., *"Where are relational databases and indexing covered?"*).
   - The AI engine executes hybrid vector search (BM25 + Dense Vectors) against Weaviate vector collections filtered strictly by the user's selected node sub-graph.
   - Results transform the canvas in real-time: matching nodes pulse with glowing borders and match badges (+1, +3), non-matching nodes dim (`opacity: 0.2`, `grayscale(1)`), connecting edges animate to illustrate knowledge flow, and the camera auto-focuses on the hit region (`fitBounds`).
   - Selecting a result opens an in-app Document Viewer that highlights parent context blocks (`SpanChunk` in yellow) and exact hit snippets (`Chunk` in orange) with smooth auto-scroll.

3. **Graph Builder & Debounce Auto-Save**:
   - Pro subscribers can build custom knowledge graphs from scratch or edit existing ones via the Gear Edit icon (`⚙`) located between the Graph Selector and "Create New" button.
   - Features drag-and-drop edge connections between right source handles and left target handles, node toolbars (`+ Add Child Node`), auto-layout engine (Dagre), and immediate source document uploads on file selection.
   - Uses a Debounced Auto-Save pattern (1000ms idle delay) to sync edits to PostgreSQL `JSONB` without locking the canvas. Draft graphs display a gear icon (`⚙`) in the selector.

4. **Product-Led Growth (PLG) & Entitlements**:
   - **Anonymous Guest**: 3 queries/day, max 2 selected nodes per search, read-only demo graphs, Better Auth anonymous session cookie, and IP abuse rate limit (max 10 anonymous sessions per IP per hour).
   - **Free User**: 20 queries/day, up to 3 custom graphs (max 10 nodes, 3 sources/node, 2MB each), max 10 selected nodes for search, standard queue priority.
   - **Pro Subscriber**: Unlimited queries (1000/month cap), unlimited custom graphs, custom document upload (PDF/MD/TXT), high RabbitMQ ingestion & query priority, Stripe/LemonSqueezy billing sync.
   - **Predefined System Graphs**: Official pre-baked templates (Computer Science, English Learning, Medicine) assigned to system user `"jbed94"`, readable by all users, editable exclusively by owner `"jbed94"`.

---

## Core Architectural Blueprint

The application is structured as a multi-repository monorepo workspace comprising five primary sub-repositories:

```
viacarraria/
├── viacarraria-frontend/       # React + React Flow + Zustand + Framer Motion Canvas UI & Decoupled Admin Panel
├── viacarraria-backend/        # Node.js (NestJS) API + CASL + Redis + Weaviate Client + Admin API
├── viacarraria-database/       # PostgreSQL + Prisma ORM + Migrations & Seeding
├── viacarraria-services/       # Python Ingestion Worker (Docling) + TEI Gemma Embeddings
└── viacarraria-infrastructure/ # Docker Compose + Helm Charts + KEDA / Karpenter Autoscaling
```

### Sub-Repository Reference Links (One-Way Submodule Mapping)

- [docs/application.prompt.md](file:///home/jbed94/Projects/viacarraria/docs/application.prompt.md) — Comprehensive application specification, UX/UI state machine, PLG rules, microservices ingestion pipeline, and agent directives.
- [viacarraria-backend/AGENTS.md](file:///home/jbed94/Projects/viacarraria/viacarraria-backend/AGENTS.md) — Backend API, Auth, CASL permissions, Rate Limiting, Weaviate integration.
- [viacarraria-database/AGENTS.md](file:///home/jbed94/Projects/viacarraria/viacarraria-database/AGENTS.md) — Relational & JSONB schemas, Prisma ORM, migrations, and template seeding.
- [viacarraria-frontend/AGENTS.md](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/AGENTS.md) — React Flow Canvas engine, UX state machine, Framer Motion, Spatial GraphRAG visualization, Decoupled Admin Panel.
- [viacarraria-services/AGENTS.md](file:///home/jbed94/Projects/viacarraria/viacarraria-services/AGENTS.md) — Python FastStream document parsing worker, Docling engine, TEI Gemma embeddings, dynamic batching.
- [viacarraria-infrastructure/AGENTS.md](file:///home/jbed94/Projects/viacarraria/viacarraria-infrastructure/AGENTS.md) — Docker Compose local dev topology, Helm K8s manifests, KEDA/Karpenter scale-to-zero, Mamba env, pre-commit config.

---

## Submodule Structural Conventions & Environment Rules

All submodules in the monorepo follow strict directory structure and environment isolation guidelines:

### 1. Minimal Mamba Environment Isolation (`environment.yml`)
- Every submodule contains an `environment.yml` defining its dedicated Mamba environment (`viacarraria-<subrepo>`).
- `environment.yml` contains **ONLY minimal platform-level tools** (e.g., Python submodules specify ONLY `python` and `poetry`; Node/TS submodules specify `nodejs` and `pnpm`/`npm`).
- Application dependencies are installed inside the Mamba environment using native package managers:
  - Python repos: `mamba run -n viacarraria-<subrepo> poetry install`
  - Node repos: `mamba run -n viacarraria-<subrepo> pnpm install`

### 2. Python Package Structure & Poetry Wrapping
- Every Python submodule MUST be wrapped with Poetry using `pyproject.toml`.
- Package source code MUST be placed inside a source folder named identically to the package (e.g. `viacarraria_services/`).

### 3. Submodule Directory Standard (`platforms/`)
- Every submodule MUST store Dockerfile definitions under `platforms/docker/`; root-level `docker/` directories are not used.
- Application sub-repositories (`frontend`, `backend`, `database`, and `services`) MUST contain only `platforms/docker/` for platform packaging.
- Only `viacarraria-infrastructure` owns the orchestration platforms `platforms/docker-compose/` and `platforms/helm/`, alongside its `platforms/docker/` directory.

---

## Key System Concepts & Data Flow

1. **Document Ingestion Pipeline**:
   - Node.js API receives file uploads and publishes jobs to RabbitMQ `document_parsing_queue`.
   - Async Python worker in `viacarraria-services` (FastStream + Docling) parses documents into Parent Chunks (~1000 tokens) and Child Chunks (~200–300 tokens).
   - HuggingFace Text Embeddings Inference (TEI) service generates Gemma Embeddings via hardware dynamic batching.
   - Chunks are stored in Weaviate using `Source` $\to$ `SpanChunk` $\to$ `Chunk` structure with Multi-Tenancy (1 tenant per graph) and Product Quantization (PQ) vector compression.
   - Ingestion status updates in Redis (`JOB_<id>:PROGRESS`) and streams to the React UI via WebSockets.

2. **Admin & Content Management**:
   - Decoupled Admin Panel (`viacarraria-frontend/dist-admin`) connecting to backend admin API provides CRUD administration, user role edits (`FREE` $\to$ `PRO`), template management (`GraphTemplate`), retention sweeps, real-time audit logging, and S3 cold storage archives with in-browser inspection.
