# Via Carraria — Application Specification

Via Carraria is an interactive visual knowledge organization, career roadmap, and skill acquisition platform. It transforms complex educational domains (e.g., computer science, medicine, language learning) into interactive Directed Acyclic Graphs (DAGs) powered by Spatial GraphRAG (Visual Retrieval-Augmented Generation).

---

## 1. Frontend

### 1.1 Technology Stack

| Layer | Technology |
|---|---|
| Language | TypeScript |
| Framework | React |
| Canvas Engine | React Flow (full-screen, `z-index: 0`, `width: 100%`, `height: 100%`) |
| Graph Layout | Dagre (Left-to-Right direction) |
| UI Components | shadcn/UI |
| Styling | Tailwind CSS + clsx |
| State Management | Zustand |
| Animations | Framer Motion (card morphing, expand/collapse transitions) |
| Notifications | Sonner (action toast notifications) |
| Internationalization | Dynamic i18n library (e.g., react-i18next or similar) |

### 1.2 Interface Architecture

The interface uses a two-layer rendering model:

1. **Canvas Layer** (`z-index: 0`): Full-width, full-height React Flow canvas displaying the knowledge DAG.
2. **Overlay Layer** (above canvas): All non-canvas UI elements — avatar icon, query field, graph selector, history panel, etc.
3. **Popup Layer** (above overlay): Maximized node detail view, document viewer, signup/subscription dialogs.

Constraint: popup views must never stack on top of each other. Opening a new popup replaces the current one.

### 1.3 Interface Modes (Zustand State Machine)

The canvas operates in three mutually exclusive modes:

The graph header displays the caller's permission (`Owner` or `View only`), a
numeric known-user access count without exposing names, and a copy action that
creates a private owned clone subject to graph quotas.

| Mode | Trigger | Behavior |
|---|---|---|
| **Idle** | Default state | Standard graph browsing, node expansion, source viewing. |
| **Context Selection** | User starts typing in query field OR presses selection toggle switch | Nodes and sources display checkboxes. User can select/deselect individual nodes and individual sources within nodes. Deselecting a node deselects all its sources. Partially selected nodes show an indeterminate checkbox (horizontal dash). |
| **Visual Results** | User clicks a completed query record in history list | Canvas transforms to highlight matching nodes. Non-matching nodes dim (`opacity: 0.2`, `filter: grayscale(1)`). Matching nodes display badge bubbles with match counts (e.g., `+1`, `+3`, `+5`). |

### 1.4 Query Field

- Position: bottom-center of overlay layer.
- Appearance: search bar styled similarly to the Android/Gemini query input pattern.
- Behavior: typing in the field transitions the interface to Context Selection mode (if not already active).
- Registered users can enable Extended Search. The API uses vectors of direct matches to query only immediately adjacent graph nodes, returning blue extended-context chunks nested beneath orange direct matches. Free users receive up to 3 extended chunks per query; Pro users receive up to 15; anonymous users cannot enable it.
- Hovering or keyboard-focusing the Extended control displays its behavior and context-budget description. Collapsing node details does not autofocus or move the camera.
- Visual-results mode includes a right-side source list that presents sources, direct matches, and optional extended context as a double nested hierarchy. Every source, direct-match, and extended-context row opens the existing source viewer with the appropriate focus and highlight color.

### 1.5 Node Interaction

**Collapsed State**: Standard DAG node.

**Expanded State** (click to toggle):
- Fixed-size card rendered on the canvas.
- Contains: markdown description + scrollable source list (PDF, TXT, MD files).
- Content scrolls within the fixed-size container (no container resize).
- Multiple nodes can be expanded simultaneously on the canvas.
- Maximize button opens the same content in a full-screen popup view (Popup Layer).

**Source Viewer**:
- Clicking a source opens a document viewer popup.
- If a maximize popup is already open, the source viewer replaces it (no stacking).
- Source opening/closing is a frequent user action. Optimize with caching or pre-fetching.

### 1.6 Query Submission & Results

1. User submits query from Context Selection mode.
2. A new record is appended to the history list with a progress bar (query in progress).
3. Interface returns to Idle mode immediately after submission.
4. User enters Visual Results mode by clicking the completed query record in history.
5. In Visual Results mode:
   - Matching nodes display badge bubbles with match counts.
   - Expanding a matching node reveals its source list sorted by maximum match score (descending).
   - Each source with matches shows a nested list of individual match snippets.
   - Clicking a source opens the document viewer focused on the highest-scoring match.
   - Clicking a specific match opens the document viewer focused on that match.
6. Document viewer highlights:
   - Parent context blocks (`SpanChunk`): yellow background.
   - Exact match snippets (`Chunk`): orange background.
   - Auto-scroll to focused match.

### 1.7 History Panel

- Position: overlay layer, bottom-left of screen.
- Displayed as a list view, visible in all three modes.
- Each history record represents a past query with its results.
- Features:
  - Click a record → enter Visual Results mode for that query.
  - Edit button → open Context Selection mode pre-populated with that query's node/source selections.
  - Pin and rename queries for organization.

### 1.8 Authentication-Dependent UI

| Element | Anonymous | Registered |
|---|---|---|
| Avatar area | "Sign Up" button → opens signup popup with standard registration form | Avatar icon → dropdown menu: Profile page, Subscription page, Logout |
| History | Not available | Full history with pin/rename |
| Query submission | Server may return anonymous quota exceeded error with signup prompt | Normal operation within tier limits |

### 1.9 Graph Selector & Management

Located in the top-left corner of the overlay layer:

1. **Graph Selector** (select/dropdown): Lists all graphs accessible to the current user. Switching selection loads the corresponding graph on the canvas. Draft (unsaved) graphs are marked with a gear icon (`⚙`). Clicking a draft graph opens it directly in edit mode.
2. **Gear Edit Icon** (`⚙`): Between the Graph Selector and "Create New" button. Opens edit mode for the currently selected graph (only for graphs the user has edit permissions on).
3. **"Create New" Button**: Opens the graph builder page.
   - Pro subscriber → navigates to the graph builder.
   - Free user → popup dialog informing about Pro subscription requirement, with link to subscription page.
   - Anonymous user → popup dialog prompting registration and subscription purchase.

### 1.10 Graph Builder / Editor

A dedicated page with the same canvas + overlay layout as the main view, but for graph construction:

- Starts with an empty canvas (new graph) or the existing graph (edit mode).
- Node operations:
  - Add an unconnected node.
  - Add a child node from an existing node.
  - Connect two existing nodes (directed edge: source → target).
  - Delete a node (confirmation dialog warning that all associated sources will also be deleted).
- Node detail view:
  - Same expanded card as in main view.
  - Scrollable source list with "Add Source" button at the bottom.
  - Each source has a delete button.
  - Source file upload happens immediately on file selection (not deferred to graph submission).
- **Debounce Auto-Save**: Graph structure is automatically saved to the database with a 1000ms idle debounce delay. This is a background draft save, separate from the explicit "Submit" action.
- **Submit Button** (bottom-center): Finalizes the graph and triggers backend preparation (document parsing, chunking, embedding). During editing, already-prepared sources are skipped (no redundant re-processing).
- **Preparation Status**: Reflected in the Graph Selector (e.g., gray dot or progress indicator similar to query history).

### 1.11 Graph Permissions

- Users can edit their own graphs by default.
- Predefined system graphs (Computer Science, English Learning, Medicine) are owned by user `"jbed94"` and are read-only for all other users.
- Only `"jbed94"` can edit system graphs.

### 1.12 Profile & Subscription Pages

**Profile Page**: Login, email, password change.

**Subscription Page**:
- Displays subscription tier cards with the current plan highlighted.
- Each card lists its feature set and limits.
- Three tiers: Free, Plus, Premium.
- Selecting a new tier opens a payment form popup (name, address, billing details — minimum required for payment processing).
- On form submission, user is redirected to the external payment system.

### 1.13 UX & Performance Requirements

- **Framer Motion**: Card expand/collapse morphing transitions.
- **React Flow**: Camera navigation, pan, zoom across the graph.
- **Sonner**: Action notification toasts.
- **Optimistic mutations**: Immediate UI feedback before server confirmation.
- **Skeleton states**: Pulsing placeholder animations while awaiting query results.
- **Micro-interactions**: Smooth hover states, transition effects via Tailwind CSS + clsx.
- **Hardware acceleration**: Use GPU-accelerated CSS transforms where beneficial.
- **Non-blocking canvas**: Canvas interactions must never lag or freeze.
- **Spatial awareness**: Smooth animated zoom/pan to matching nodes in Visual Results mode (`fitBounds`).
- **Edge animation**: When two connected nodes both have matches, animate the connecting edge (e.g., highlighted color) to show knowledge flow.
- **Auto-focus on results**: Automatically zoom and center the viewport on the graph region containing matches.

---

## 2. Backend

### 2.1 Technology Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js |
| Framework | NestJS |
| ORM | Prisma |
| Database | PostgreSQL |
| Vector Database | Weaviate |
| Cache / Session | Redis |
| Task Queue | RabbitMQ |
| Auth Permissions | CASL (ability-based authorization) |
| Payments | Stripe or LemonSqueezy (prefer LemonSqueezy for international tax/billing) |

### 2.2 Data Storage

**PostgreSQL (via Prisma ORM)**:
- Users, subscriptions, roles, session state.
- Graph definitions stored as `JSONB` columns with `nodes` and `edges` arrays (fast read/write for graph selector switching).
- Graph metadata: ownership, visibility (public/private/shared), preparation status.
- Dedicated model for graphs with additional attributes (title, owner, visibility, status, timestamps).

**Weaviate (Hybrid Vector Database)**:
- Stores parsed document representations using a three-level hierarchy:
  - `Source`: Original document reference (PDF, TXT, MD).
  - `SpanChunk`: Parent context block (~1000 tokens). Contains multiple consecutive Chunks.
  - `Chunk`: Atomic text segment (~200–300 tokens). Each Chunk has an embedding vector.
- Hybrid search (BM25 + Dense Vector) executes against `Chunk` objects.
- Search results return the parent `SpanChunk` as display context, with the matching `Chunk` highlighted within it.
- All chunks and span chunks store position/location references within the original source document for precise highlighting in the document viewer.
- **Multi-Tenancy**: One Weaviate tenant per knowledge graph (graph-level data isolation).
- **Product Quantization (PQ)**: Vector compression to reduce storage.
- Design note: chunk text content is stored in Weaviate, but position metadata (offsets, page numbers) is also stored to enable accurate highlighting in the original document.

**Redis**:
- Anonymous session keys with TTL (24h).
- Ingestion job progress tracking (`JOB_<id>:PROGRESS`).
- Query rate limiting counters.
- Task deduplication.
- Inter-service communication with Node.js backend.

### 2.3 Weaviate Integration from Backend

The Node.js backend manages Weaviate for:
- Creating `Source` objects when files are uploaded.
- Executing hybrid search queries filtered by tenant (graph) and selected node sub-graph.
- Reading search results.

The backend does NOT create `SpanChunk` or `Chunk` objects — that is handled by the Python ingestion services.

### 2.4 Product-Led Growth (PLG) & Rate Limiting

**Tier Entitlements**:

| Capability | Anonymous Guest | Free User | Pro Subscriber |
|---|---|---|---|
| Queries/day | 3 | 20 | Unlimited (1000/month cap) |
| Max selected nodes per search | 2 | 10 | Unlimited |
| Custom graphs | 0 (read-only demo graphs) | 3 (max 10 nodes each, 3 sources/node, 2MB/source) | Unlimited (higher node/source limits) |
| Document upload | No | Yes (within limits) | Yes (PDF/MD/TXT) |
| Queue priority | Standard | Standard | High (RabbitMQ priority) |

**Anonymous Session Management**:
- Better Auth anonymous users receive an opaque HttpOnly session cookie.
- Anonymous identity and session records are stored by Better Auth in PostgreSQL.
- IP-based abuse prevention: max 10 anonymous sessions per IP per hour.

**Rate Limiting Strategy** (three layers):
1. **Rate Limit**: Requests per time window (applies to all tiers including Pro, to prevent abuse like 1000 machines × 1000 concurrent queries).
2. **Usage Quota**: Daily/monthly query budget per tier.
3. **Complexity Limit**: Constraints on query scope (selected nodes, source count).

Implementation: `@upstash/ratelimit` or custom NestJS middleware. Feature gating via CASL ability definitions.

**Payments**: Stripe or LemonSqueezy integration. LemonSqueezy preferred for simplified international tax handling and multi-market expansion.

---

## 3. Ingestion Microservices

### 3.1 Architecture Overview

The ingestion pipeline is a set of Python-based async workers that parse documents, generate embeddings, and write results to Weaviate.

### 3.2 Technology Stack

| Component | Technology |
|---|---|
| Task Broker | RabbitMQ (reliable delivery, DLX, DLQ, priority queues) |
| Inter-service state | Redis (progress, deduplication, communication with Node.js) |
| Document Parser | Docling |
| Embedding Model | Gemma Embedding (latest version) |
| Embedding Service | HuggingFace TEI (Text Embeddings Inference) |
| Worker Framework | FastStream + RabbitBroker |

### 3.3 Pipeline Flow

1. Node.js backend publishes a parsing job to RabbitMQ `document_parsing_queue`.
2. Python worker (FastStream + RabbitBroker) consumes the job:
   a. Parses the document using Docling into `SpanChunks` (~1000 tokens) and `Chunks` (~200–300 tokens).
   b. Sends async embedding requests to the TEI service for each Chunk.
   c. Writes `SpanChunk` and `Chunk` objects (with embedding vectors) to Weaviate.
   d. Updates job progress in Redis (`JOB_<id>:PROGRESS`).
3. Node.js backend streams progress updates to the React frontend via WebSockets.

### 3.4 Dynamic Batching

- Embedding generation via TEI must use dynamic batching (max batch size + max wait time) for efficient GPU/CPU utilization.
- Document parsing may also benefit from dynamic batching if the parser supports it.

### 3.5 Queue Prioritization

RabbitMQ priority levels (descending):
1. System/admin tasks.
2. Pro subscriber tasks.
3. Free user tasks.
4. Anonymous guest tasks (if applicable).

---

## 4. Infrastructure

### 4.1 Local Development (Docker Compose)

Split into two compose profiles:

| Profile | Services |
|---|---|
| **infra** | PostgreSQL, Weaviate, RabbitMQ, Redis |
| **app** | Frontend, Backend (NestJS), Python worker(s), TEI embedding service |

### 4.2 Production Deployment (Kubernetes + Helm)

Helm chart with the following scaling configuration:

| Component | Scalable | Scale to Zero | Notes |
|---|---|---|---|
| Frontend | Optional | No | Static or minimal replicas |
| Backend (NestJS) | Yes | No | Always-on for API availability |
| Python Worker | Yes | Yes | Scale with KEDA based on RabbitMQ queue depth |
| TEI Embedding | Yes | Yes | Scale with KEDA based on embedding queue depth |
| PostgreSQL | No (initially) | No | Single instance; evaluate later |
| Weaviate | Evaluate later | No | May need scaling for large deployments |
| Redis | No (initially) | No | Single instance; evaluate later |

**Scaling tools**:
- **KEDA**: Pod-level autoscaling (add/remove pods based on queue metrics).
- **Karpenter**: Node-level autoscaling (provision/deprovision machines).

**GPU Strategy**: Deploy embedding services on CPU initially. Gemma Embedding models are ~300M parameters — CPU handles them adequately. All ingestion requests are async, so latency is acceptable. Maintain a ready-to-deploy GPU configuration for future scaling.

**Helm Requirements**:
- Secret management (Kubernetes Secrets or external vault).
- RBAC and access controls.
- Horizontal pod autoscaling definitions.
- Ingress configuration for all services.

### 4.3 Admin Panel (Directus)

- Self-hosted Directus instance mounted on the same PostgreSQL database.
- Provides CRUD admin UI for: user management, role assignment (`FREE` → `PRO`), graph template management, JSONB graph inspection/editing.
- In Kubernetes: deployed on a separate subdomain, secured at the ingress level (IP allowlist, basic auth, or SSO).
- Enables password resets, role changes, and data inspection without custom admin backend code.

---

## 5. Cross-Cutting Concerns

### 5.1 Environment Management (Mamba) & Submodule Architecture

- **Minimal Tooling Mamba Environments**:
  - Every sub-repository has its own `environment.yml` defining a dedicated Mamba environment (`viacarraria-<subrepo>`).
  - Mamba environments contain **ONLY minimal platform-level tools** (e.g., Python repositories specify ONLY `python` and `poetry`; Node/TS repositories specify `nodejs` and `pnpm`/`npm`).
  - Application dependencies are managed and installed exclusively via language-native package managers inside Mamba:
    - Python repositories: `mamba run -n viacarraria-<subrepo> poetry install`
    - Node repositories: `mamba run -n viacarraria-<subrepo> pnpm install`
  - All developer and CI commands (run, test, lint, format) must be executed via `mamba run -n viacarraria-<subrepo> <command>`.

- **Python Package Wrapping & Layout**:
  - Every Python sub-repository MUST be Poetry-wrapped with `pyproject.toml`.
  - Package source code MUST be placed inside a source folder named identically to the package (e.g. `viacarraria_services/`).

- **Submodule Container & Platform Standardization (`platforms/`)**:
  - Every submodule MUST store Dockerfile specifications under `platforms/docker/`; root-level `docker/` directories are not used.
  - Application sub-repositories MUST NOT contain `platforms/docker-compose/` or `platforms/helm/`.
  - Only `viacarraria-infrastructure` owns `platforms/docker-compose/` and `platforms/helm/`, alongside `platforms/docker/`.

### 5.2 Pre-Commit Hooks

Every sub-repository must have a `.pre-commit-config.yaml` with appropriate hooks (linting, formatting, type checking). Hooks must execute commands inside the Mamba environment (`mamba run`).

### 5.3 Internationalization

The frontend must support language switching using a dynamic i18n library. All user-facing strings must be externalized into translation files.

---

## 6. Agent AI Behavioral Directives

The following directives govern how an AI agent should operate when working on this codebase.

### 6.1 Code Quality

- Write correct code on the first attempt. Do not rely on iterative linter-fix loops.
- Run formatting once (e.g., `mamba run -n <env_name> ruff format`). Do not repeatedly invoke the agent to fix formatting or lint warnings.
- Leave residual warnings and non-critical errors for the user to review. Do not enter fix-retry loops.

### 6.2 Testing Strategy

- Write tests for created functionality. Tests are a verification tool, not the primary deliverable.
- Keep test complexity proportional to the feature being tested. Do not over-engineer test suites.
- Do not start infrastructure or the full application stack to test individual features. Write targeted unit/integration tests instead.
- Use infrastructure or full-stack startup only for: user-reported issues, end-to-end verification, or behaviors that cannot be tested in isolation.

### 6.3 Context & Planning Discipline

The agent must maintain a continuous working loop that:
1. **Manages context**: Keeps context precise, current, and persistent across sessions. Monitors context window length and compacts when needed.
2. **Maintains a plan**: Creates and updates a structured task plan. New sub-tasks discovered during execution must be added to the plan in real-time.
3. **Tracks progress**: Records completed steps, blockers, and decisions as work proceeds.
