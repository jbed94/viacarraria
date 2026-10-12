# Via Carraria — Comprehensive UI/UX Simplification & Aesthetics Architectural Report

**Target Document**: `docs/ui-ux-design-report.md`  
**Workspace**: Via Carraria Monorepo  
**Submodules Audited**: [`viacarraria-frontend`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend)  
**Core Objective**: Deconstruct current UI/UX friction points, eliminate visual clutter and cognitive overload, establish strict progressive disclosure, transition in-canvas data bloat into dedicated detail views, and provide a unified, aesthetic design blueprint for production.

---

## 1. Executive Summary

Via Carraria offers an innovative paradigm: converting complex disciplines into Left-to-Right (LR) Directed Acyclic Graphs (DAGs) powered by Spatial GraphRAG search and topological crawling. The canvas is intended to serve as a spatial visual map of knowledge.

However, iterative feature additions have introduced severe **visual clutter**, **cognitive overload**, and **architectural layout friction**:
1. **The "Everything Everywhere" Antipattern**: Rather than utilizing progressive disclosure, the interface attempts to render all information simultaneously. Individual canvas nodes expand in-place into massive 500px cards containing Markdown note editors, multi-stream file upload queues, and full search snippet lists. When multiple nodes expand, the spatial DAG is obscured by overlapping boxes.
2. **The Search Bar "Swiss Army Knife"**: The floating search dock houses a query input, a hover-triggered mode switcher, a batch node selection manager, an autocomplete popover with three tiers of chips, and an accordion drawer with 5+ numeric sliders (Digs, Links, Jumps, Matches per Step, Scope, Direction) plus a multi-hypothesis group cycling matrix.
3. **Fragmented Dual-Canvas Model**: Performing a topological crawl replaces the user's primary graph with `CrawlResultsCanvas`, a completely separate 2900-line React Flow instance equipped with over 12 distinct status pills, filter toggles, diff view switches, and export dropdowns crowded into the top 70 pixels of the viewport.
4. **Header Island Fragmentation & Layout Shifts**: The top navigation is divided into three separate floating islands that appear, disappear, and jump horizontally as graph permissions change, crowded with native HTML `<select>` dropdowns, storage meters, deletion warnings, active viewer counters, and language flags.
5. **Color & Animation Satiation**: The canvas displays up to 10 competing saturated colors (seed blue, dig emerald, link amber, jump purple, 6 hypothesis palette tones, coral hit badges, gold highlights) alongside simultaneous looping animations (pulsing logo, pulsing streaming indicator, glowing node borders, animated dashed edge flows).

### The Core Design Principle: The Semantic Canvas & Dedicated Detail Views

The central thesis of this report is:
> **The canvas is a spatial map for mental orientation, not an accordion of text editors. Canvas nodes must remain lightweight semantic tokens. All deep inspection, document editing, parameter tuning, and citation exploration must live in dedicated, focused views (Slide-Out Drawers, Focused Readers, and Dedicated Studios).**

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                 BEFORE: IN-PLACE CANVAS CLUTTER                                 │
├─────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                 │
│    [Top Bar: 12 buttons/badges across 3 floating islands with layout shifts]                   │
│                                                                                                 │
│    ┌───────────────┐          ┌─────────────────────────────────────────────────────┐           │
│    │  Small Node   │─────────▶│             IN-PLACE EXPANDED NODE (500px)          │           │
│    └───────────────┘          │ - Edit Title & Category Input                       │           │
│                               │ - Description Textarea                              │           │
│    ┌───────────────┐          │ - Live Upload Queue (Pause/Resume/Streams/%)        │           │
│    │  Small Node   │          │ - New Note Form (Title + Body + Save)               │           │
│    └───────┬───────┘          │ - 8 Match Snippets with nested source lists         │           │
│            │                  └─────────────────────────────────────────────────────┘           │
│            ▼                                    (Overlaps edges and adjacent nodes!)            │
│    ┌───────────────┐                                                                            │
│    │  Small Node   │                                                                            │
│    └───────────────┘                                                                            │
│                                                                                                 │
│    [Search Dock: Query + 5 Sliders + 3 Suggestion Chip Rows + Hypothesis Group Cycler]         │
└─────────────────────────────────────────────────────────────────────────────────────────────────┘

                                                ▼ REFACTORED TO

┌─────────────────────────────────────────────────────────────────────────────────────────────────┐
│                          AFTER: SEMANTIC CANVAS + DEDICATED INSPECTOR                           │
├─────────────────────────────────────────────────────────────────────────────────────────────────┤
│  [Top Bar: Logo / Breadcrumb Popover]                                [⌘K] [Bell] [Avatar Hub]   │
│                                                                                                 │
│  ◀── SPATIAL KNOWLEDGE CANVAS (Clean, Uncluttered) ──▶ │ ◀── DEDICATED TOPIC INSPECTOR ──▶    │
│                                                        │     (Slide-Out Right Drawer)           │
│    ┌──────────────┐          ┌──────────────┐          │ ┌────────────────────────────────────┐ │
│    │  Dijkstra    │─────────▶│   A* Search  │          │ │  Topic: Dijkstra Algorithm         │ │
│    │  📄 2  ✦ 3   │          │   📄 1  ✦ 0  │          │ │  [Overview] [Sources] [Matches]    │ │
│    └──────────────┘          └──────────────┘          │ ├────────────────────────────────────┤ │
│           │                         │                  │ │ Syllabus & Study Notes             │ │
│           ▼                         ▼                  │ │ Description, formula, complexity.  │ │
│    ┌──────────────┐          ┌──────────────┐          │ ├────────────────────────────────────┤ │
│    │  Fibonacci   │─────────▶│ Shortest Path│          │ │ Attached Sources (2 PDFs)          │ │
│    │  📄 1  ✦ 1   │          │   📄 3  ✦ 4  │          │ │ Upload File Dropzone               │ │
│    └──────────────┘          └──────────────┘          │ ├────────────────────────────────────┤ │
│                                                        │ │ Search Hits & Citations            │ │
│  [Floating Search Pill: 🔍 Search / 🧭 Crawl | Input ⌘K | ⚙ Tune] │ │ Exact snippets & page references   │ │
└─────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. In-Depth UX Heuristic Audit (Component-by-Component)

### 2.1 The Topbar & Navigation System (`header.tsx`)

#### Current Architecture
- Located in [`viacarraria-frontend/src/components/overlays/header.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/overlays/header.tsx).
- Divided into three separate floating divs:
  1. `topbar-zone-left`: Logo, brand title, native HTML `<select>` graph picker, Public/Private badge, active viewer count pill, and "⚠️ Deletion Scheduled" warning badge.
  2. `topbar-zone-center`: Contextual button island (`topbar-attach-btn`, `Settings2` edit button, `Sliders` settings button). Hidden via CSS `:empty { display: none }`.
  3. `topbar-zone-right`: Limits Gauge button (opens giant popup), Notifications bell, Theme toggle, Language flags dropdown, User avatar button.

#### Usability & Aesthetic Flaws
1. **Layout Shifts (Jank)**: Because `topbar-zone-center` renders conditionally based on `graph.canEdit` or `isUnattachedPublic`, navigating between owned and public graphs causes the entire topbar to jump horizontally.
2. **Native `<select>` Friction**: The graph picker uses a native browser `<select>` with `<optgroup>` elements. It cannot be styled consistently across operating systems, cannot display search filters, icons, or preview badges, and looks out of place inside a floating glassmorphic shell.
3. **Badge Satiation**: A single graph title can be flanked by: `[Draft]`, `[Public]`, `[👁 3]`, and `[⚠️ Deletion Scheduled]`. This creates high visual noise before the user even looks at the graph.
4. **Action Hierarchy Inversion**: The Gauge icon (Limits) opens a panel detailing storage megabytes, query counts, and node caps. This is account-level administration, not a high-frequency action needed on the canvas header.
5. **Language Flags Clutter**: Displaying national flags (`🇬🇧`, `🇵🇱`, `🇪🇸`, `🇮🇹`) on the topbar takes up horizontal space that mobile and tablet viewports cannot spare.

---

### 2.2 The Search & Crawl Dock (`search-bar.tsx`)

#### Current Architecture
- Located in [`viacarraria-frontend/src/components/overlays/search-bar.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/overlays/search-bar.tsx) (1,657 lines).
- Implements:
  - Floating Query Hints row above the bar (`crawl-seed-chip` warnings and counts).
  - Left mode switcher column with hover dropdown (`showModeMenu` on mouse enter).
  - Main search input with keyboard event handlers (`ArrowUp`, `ArrowDown`, `Enter`, `Escape`).
  - Autocomplete popover with three categorized chip rows: Suggested Queries, Node Concepts, and Concept Clusters, plus a link to the Vocabulary Cloud dialog.
  - Slide-out options drawer (`search-options-drawer`) containing:
    - Search Scope `LineSlicer` (`Normal`, `Deep`, `Graph-wide`).
    - Crawl Direction `LineSlicer` (`Forward`, `Backward`, `Bidirectional`).
    - 4 distinct `LineSlicer` widgets for crawl exploration limits: Digs (0..5), Links (0..5), Jumps (0..5), and Matches per Step (1..5).
    - Comparative Mode toggle switch with hypothesis group cycling pills (`groupA` through `groupF`).

#### Usability & Aesthetic Flaws
1. **Severe Violation of Progressive Disclosure**: A user wanting to ask a simple question is confronted with a massive control panel containing algorithm hyperparameters (digs, links, jumps, steps, scopes, directions, comparative partitions).
2. **Hover Menu Ergonomic Failure**: The left mode switcher uses `onMouseEnter` / `onMouseLeave`. Users moving their pointer towards the input accidentally trigger the mode menu, or try to select a mode and have the menu vanish when their cursor strays 2 pixels outside the boundary.
3. **Floating Hints Row Layout Conflict**: When switching to crawl mode, the `search-bar-hints-row` pops up above the bar, pushing the search bar downwards and causing the canvas viewport behind it to jitter.
4. **Autocomplete Popover Overcrowding**: The suggestions popover renders query chips, vocabulary chips, and concept cluster chips simultaneously. It occupies up to 400px of height and covers the bottom third of the knowledge graph.

---

### 2.3 Canvas Nodes & In-Place Expansion Bloat (`subject-node.tsx`)

#### Current Architecture
- Located in [`viacarraria-frontend/src/components/canvas/subject-node.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/canvas/subject-node.tsx) (788 lines).
- Implements two render states:
  - **Collapsed**: Node title, category tag, handles, selection checkbox, source count indicator.
  - **Expanded**: Dynamically expands in-place directly on the canvas into a 450px $\times$ 500px card containing:
    - Editable Title input and Category input.
    - Editable Description textarea.
    - Matches list with nested source buttons and clickable text snippet blocks.
    - Sources list with active upload tracks (circular progress, pause button, resume button, cancel button, concurrency pills, percent badges).
    - New Note creation form (title input, body textarea, save/cancel buttons).
    - Header action buttons (Maximize to modal, Add child, Delete, Collapse).

#### Usability & Aesthetic Flaws
1. **Destruction of the Mental Map**: A graph canvas is designed for spatial relationship discovery. Expanding a node in-place drastically alters its dimensions, occludes neighboring nodes, cuts through connecting edges, and forces the user to zoom out, losing their reading focus.
2. **Laggy Canvas Panning & Re-flow**: When multiple nodes expand, rendering heavy HTML forms, scrollable lists, and file upload progress rings inside React Flow's zoomable SVG/HTML canvas causes noticeable frame drops on large graphs.
3. **Duplication of Responsibility**: There are currently three distinct places to view or edit a node:
   - In-place expanded card on canvas ([`subject-node.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/canvas/subject-node.tsx)).
   - Maximized modal dialog ([`node-detail-modal.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/dialogs/node-detail-modal.tsx)).
   - Node editor dialog ([`node-editor-dialog.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/dialogs/node-editor-dialog.tsx)).
   This creates triple maintenance overhead and confuses users as to which interface is the canonical editor.

---

### 2.4 The Crawl Results Canvas Disconnect (`crawl-results-canvas.tsx`)

#### Current Architecture
- Located in [`viacarraria-frontend/src/components/canvas/crawl-results-canvas.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/canvas/crawl-results-canvas.tsx) (2,940 lines).
- When a crawl query finishes, `App.tsx` unmounts `GraphCanvas` and mounts `CrawlResultsCanvas`.
- Implements:
  - Top header toolbar with: Query title, Info button, Streaming status pill, 3-button Diff View Switcher (`Single Focus`, `Unified Flow`, `Split Diff`).
  - Summary pills row: Seed pill, Digs pill, Links pill, Jumps pill, and Total Matches pill.
  - Active Trail spotlight chip with Clear button.
  - Keyword Filter chip with Clear button.
  - Traversal Filter button.
  - Export Trail dropdown.
  - Multiple side popovers (Traversal filter drawer, Query info panel, Subtree synthesis dialog, Return confirmation modal).

#### Usability & Aesthetic Flaws
1. **Sensory & Visual Overload in the Header**: The top 70 pixels of the viewport contain up to **14 interactive buttons, pills, and dropdowns** side by side. It feels like an aircraft cockpit rather than an educational research tool.
2. **Context Disorientation**: Toggling between regular search and crawl mode swaps out the entire canvas component. The user's spatial orientation (pan, zoom level, node positions) is reset, making it difficult to understand where the crawled subgraph sits relative to the overall subject tree.
3. **Over-Engineered Micro-Metrics**: Displaying exact match type counts (Digs: 3, Links: 2, Jumps: 1) as solid colored pills directly on the primary viewing bar distracts from the actual content. Users want to read the answer trajectory, not parse taxonomy counters.

---

### 2.5 Results Sidebar (`results-sidebar.tsx`) & Reading Modals

#### Current Architecture
- Located in [`viacarraria-frontend/src/components/overlays/results-sidebar.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/overlays/results-sidebar.tsx) (884 lines).
- Features:
  - Lead Answer dossier with highlighted passage, answer type badge, and prerequisite/extension graph chips.
  - Unified Citation Inspector card with match score %, page number, highlighted blockquote, "Focus Node", and "Open Document" buttons.
  - Filter mode pills (`All`, `Same Source`, `Adjacent`).
  - Source match list with snippet highlights.
  - Full-screen modal for document viewing ([`source-viewer-dialog.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/dialogs/source-viewer-dialog.tsx)).

#### Usability & Aesthetic Flaws
1. **Vertical Crowding in Sidebar**: Squeezing the Lead Answer, Citation Inspector, Filter pills, and Source lists into a single 420px vertical column forces continuous scrolling. The Citation Inspector takes up 200px of height, pushing remaining search hits below the fold.
2. **Jarring Document Modal Cover-Up**: Clicking "Open Document" opens a full-screen dialog over the entire workspace. The user loses visibility of the canvas, the search sidebar, and their spatial location. When they close the modal, they have to reorient themselves.

---

## 3. Core UX Design Patterns & Esthetic Guidelines

To transform Via Carraria into an intuitive, elegant, and focused application, the frontend must adhere to four strict design principles:

### 3.1 Progressive Disclosure (The 3-Level Depth Model)

Never expose secondary or tertiary information until the user explicitly requests it.

```
┌────────────────────────────────────────────────────────────────────────┐
│                        3-LEVEL DEPTH MODEL                             │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│  LEVEL 1: Glanceable Spatial Canvas                                    │
│  - Objective: Spatial mental map & topological orientation.            │
│  - Scope: Node title, category badge, match indicator (+3).            │
│  - No editable inputs, no text areas, no upload queues on canvas.      │
│                                                                        │
│                           │ (User clicks node)                         │
│                           ▼                                            │
│                                                                        │
│  LEVEL 2: Dedicated Inspector Drawer (Right Sheet)                     │
│  - Objective: Structured inspection & authoring.                       │
│  - Scope: Description, study notes, attached sources, upload dropzone, │
│    exact search match citations, and prerequisite connections.         │
│                                                                        │
│                           │ (User clicks citation / document)          │
│                           ▼                                            │
│                                                                        │
│  LEVEL 3: Focused Document Reader (Side-by-Side Split or Max Viewer)   │
│  - Objective: Deep reading & verification.                             │
│  - Scope: Full PDF / Markdown viewer with yellow/orange hit spans.     │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

---

### 3.2 The 60-30-10 Color & Semantic Restraint Rule

Currently, the interface suffers from "rainbow syndrome" with over 10 competing saturated hues. We replace this with a restrained, high-contrast editorial system:

- **60% Dominant Neutral**:
  - Light Mode: Warm, organic paper whites (`#FBFBF9`, `#F4F1EA`).
  - Dark Mode: Deep obsidian/slate neutrals (`#121614`, `#18201A`).
- **30% Structural Hierarchy**:
  - Cards, sidebars, borders, and typography:
    - Light: Deep forest slate (`#253229`), muted sage (`#6E786F`), delicate lines (`#E2DDD2`).
    - Dark: Elevated surfaces (`#1E2620`), soft lines (`#2E3A30`), crisp paper text (`#EDF5E9`).
- **10% Intentional Accent**:
  - Single primary brand accent: Deep Forest Mint / Emerald (`#1B663E` in light, `#4ADE80` in dark).
  - Used *only* for primary actions, current active selection, and confirmed execution.
- **Semantic Mappings**:
  - Matches & Highlights: Warm Amber (`#D97706` text, `#FEF3C7` background).
  - Prerequisites / Flow: Subtle cool slate (`#64748B`).
  - Destructive Actions: Soft terracotta crimson (`#DC2626`).
  - **No neon solid fills for badges**: Replace solid green/blue/purple badges with muted neutral badges featuring subtle 1px border tints and 11px uppercase typography.

---

### 3.3 Typography & Hierarchy Scale

Establish strict typographic roles using clean modern fonts (Inter or IBM Plex Sans):

| Role | Font Size | Weight | Line Height | Letter Spacing | Usage |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Title / Heading** | 16px | 600 | 1.25 | `-0.01em` | Node titles, panel headers |
| **Body / Lead** | 14px | 400 | 1.50 | `normal` | Study notes, lead answer text |
| **Secondary / Meta** | 12px | 500 | 1.40 | `normal` | File names, timestamps, prerequisites |
| **Micro / Badges** | 10px | 600 | 1.00 | `+0.05em` | Status tags, match indicators, counts |

---

### 3.4 Spatial Consistency & Whitespace

1. **Elevation & Depth**: Remove harsh multi-layer drop shadows (`box-shadow: 0 4px 16px rgba(0,0,0,0.12)`). Use subtle 1px border outlines combined with soft diffuse ambient shadows (`0 8px 30px rgba(0,0,0,0.04)`).
2. **Animation Discipline**:
   - Turn off continuous looping animations during idle exploration.
   - The logo, node borders, and streaming badges should *not* pulse infinitely.
   - Animations must be functional micro-interactions only: 150ms ease-out transitions on hover, smooth 300ms cubic-bezier camera focus, and subtle 200ms accordion drawer reveals.

---

## 4. Architectural Blueprints: The Simplified UI System

### 4.1 Global Viewport Wireframe Blueprint

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────┐
│ [Logo] Via Carraria / [Computer Science ▾]       [Edit Map]      [⌘K Search]  [🔔] [Avatar ▾]  │
├─────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                 │
│                                                                ┌──────────────────────────────┐ │
│         ┌──────────────┐             ┌──────────────┐          │ TOPIC INSPECTOR (SLIDE-OVER) │ │
│         │   Dijkstra   │────────────▶│  A* Search   │          ├──────────────────────────────┤ │
│         │  📄 2  ✦ 3   │             │   📄 1  ✦ 0  │          │ [Overview] [Docs] [Citations]│ │
│         └──────────────┘             └──────────────┘          ├──────────────────────────────┤ │
│                │                            │                  │ Description & Syllabus Notes │ │
│                ▼                            ▼                  │ Algorithmic formulation and  │ │
│         ┌──────────────┐             ┌──────────────┐          │ time complexity O(V log V).  │ │
│         │  Fibonacci   │────────────▶│Shortest Path │          ├──────────────────────────────┤ │
│         │  📄 1  ✦ 1   │             │   📄 3  ✦ 4  │          │ Attached Sources (2 files)   │ │
│         └──────────────┘             └──────────────┘          │ 📄 Dijkstra.pdf (Ready)      │ │
│                                                                │ 📄 PriorityQueue.md (Ready)  │ │
│                                                                │ [+ Upload Document]          │ │
│                                                                └──────────────────────────────┘ │
│                                                                                                 │
│                                                                                                 │
│                    ┌──────────────────────────────────────────────────────┐                     │
│                    │  [🔍 Search | 🧭 Crawl]  Ask a question... (⌘K)  [⚙]  │                     │
│                    └──────────────────────────────────────────────────────┘                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

### 4.2 Topbar: Unified Breadcrumb & Workspace Hub

#### Before vs. After Specification

- **Before**: 3 floating boxes, HTML `<select>` with `<optgroup>`, 4 flags, separate limits gauge, separate edit button, separate settings slider button, separate deletion pills.
- **After**:
  - **Left**: `[Brand Icon] Via Carraria / [Graph Title ▾]`
    - Clicking `[Graph Title ▾]` opens a modern popover:
      - Quick Switcher: Searchable list of attached/owned graphs with recent access dates.
      - Action menu: `Create Graph`, `Duplicate Graph`, `Graph Settings & Visibility`, `Retention & Deletion Status`.
  - **Center**: Clean, subtle graph status indicator:
    - When editing: `[Editing Mode · Auto-saved] [Done Editing]` button.
    - When exploring public graph: Single `[Bookmark & Attach]` button.
  - **Right**:
    - `[⌘K Search]` quick trigger button.
    - `[🔔]` Notifications bell (displays a simple 6px indicator dot when unread; clicking opens notifications popover).
    - `[Avatar ▾]` Consolidated Account Hub:
      - Profile information.
      - Usage & Entitlements: Registered Tier, Storage quota (e.g. `12.4 MB / 100 MB`), Daily queries remaining (`48 / 50`).
      - Preferences: Theme (`Light`, `Dark`, `System`), Language selector dropdown.
      - Retention dashboard & Privacy policy links.
      - Logout.

**Result**: Reduces topbar interactive elements from **11 down to 4**, eliminates layout shift, and clears 80px of vertical space for the canvas.

---

### 4.3 Search Bar: Clean Command Pill & Dedicated Crawl Studio

#### Before vs. After Specification

- **Before**: 1,657 lines of code in `search-bar.tsx`. Contains hover menus, multi-chip autocomplete popovers, hints row, and an accordion with 5 sliders for Digs/Links/Jumps/Steps/Direction plus hypothesis cycling pills.
- **After**:
  1. **The Floating Command Pill**:
     ```
     ┌────────────────────────────────────────────────────────────────────────┐
     │ [🔍 Search │ 🧭 Crawl] │ Query or explore concepts... (⌘K) │ [⚙] │ [↵]│
     └────────────────────────────────────────────────────────────────────────┘
     ```
     - Pill width: 560px centered at the bottom of the canvas (floating with 24px bottom margin).
     - Segmented toggle: Clean two-tab pill (`Search` | `Crawl`). No hover popover!
     - In Crawl mode: The input shows a subtle badge: `[ 🧭 1 seed selected ]` or `[ 🧭 Click a node to start ]`.
     - `[⚙]` Settings button: Opens the **Crawl & Hypothesis Studio** drawer.
  2. **Dedicated "Crawl & Hypothesis Studio" (Right Drawer / Flyout)**:
     - When users want to adjust crawl hyperparameters (Digs, Links, Jumps depth, Comparative Hypotheses A vs B), they click `[⚙]`.
     - Instead of cluttering the canvas with cramped sliders, this opens a dedicated 400px side panel titled **Traversal Configuration**:
       - Traversal Direction (`Forward Dependencies`, `Backward Prerequisites`, `Bidirectional`).
       - Crawl Depth (`Shallow - 1 hop`, `Standard - 2 hops`, `Deep - 3 hops`).
       - Match Categories: Checkboxes for Digs (Elaborations), Links (Connections), Jumps (Conceptual Bridges) with clear explanatory tooltips.
       - Comparative Mode: Clean hypothesis partition manager allowing users to assign selected nodes to Hypothesis A vs Hypothesis B with distinct color tokens.
  3. **Streamlined Autocomplete Popover**:
     - Standard Command Palette pattern:
       - Section 1: Recent Searches (max 3).
       - Section 2: Matching Topics in Active Graph (max 3, with arrow navigating camera to node).
       - Section 3: Suggested Queries (max 2).
     - Full Vocabulary Cloud & Concept Atlas moves to a dedicated modal opened via `[Explore Concept Atlas ↗]`.

---

### 4.4 Canvas Nodes: The Semantic Node Token & Dedicated Right Sheet

#### Before vs. After Specification

- **Before**: In-place expansion on the canvas. Cards grow from 200px to 500px, rendering textareas, file upload controls, note creators, and match snippet lists directly on the React Flow canvas plane.
- **After**:
  1. **The Semantic Node Card on Canvas**:
     ```
     ┌──────────────────────────────────────────────────┐
     │ TOPIC                                       [+]  │
     │ Dijkstra Shortest Path                           │
     │ ──────────────────────────────────────────────── │
     │ 📄 2 sources                   ✦ 3 search hits   │
     └──────────────────────────────────────────────────┘
     ```
     - Fixed, predictable dimensions: `width: 220px`, `height: 84px`.
     - Clean typography: Category pill (10px uppercase), Title (14px bold), bottom metadata row (11px muted).
     - Selection state: 2px accent border and subtle glow. Unselected nodes dim cleanly during search.
     - Hover micro-action: Subtle `[+]` button to add child node (in edit mode) or `[→]` to inspect details.
  2. **The Dedicated Topic Inspector (Slide-Out Right Sheet)**:
     - When any node is clicked on the canvas:
       - The canvas camera gently centers on the node.
       - A sleek 420px **Topic Inspector Sheet** slides in from the right.
       - **Tab 1: Overview & Notes**:
         - Node title and category editor (if permitted).
         - Topic description.
         - Study notes with formatted Markdown rendering.
         - `[+ Add Study Note]` button.
       - **Tab 2: Attached Sources**:
         - Document list with status pills (`Ready`, `Processing`, `Error`).
         - File upload dropzone with background upload queue.
         - Document preview and download buttons.
       - **Tab 3: Search Hits & Citations**:
         - Filterable list of all exact chunk matches and relevance percentages.
         - Click snippet to open in the Source Document Viewer.
       - **Tab 4: Topological Hierarchy**:
         - Prerequisite nodes (parents) and Extension nodes (children) with one-click navigation.

**Key Advantage**: The canvas remains an unobstructed, high-level map at all times. Users can click through 10 nodes in 10 seconds while the right Inspector smoothly updates, without any jarring canvas re-layouts.

---

### 4.5 Crawl Interface: Integrated Crawl HUD & Dedicated "Crawl Dossier"

#### Before vs. After Specification

- **Before**: 2,940 lines in `crawl-results-canvas.tsx`. Swaps out the main canvas for a separate canvas. Top bar crammed with 14 pills, counters, view switches, and export menus.
- **After**:
  1. **Single Canvas Architecture**:
     - Do *not* unmount the knowledge canvas!
     - Instead, render the crawl results as a **Topological Traversal Layer** directly on the active canvas.
     - The traversed nodes highlight with glowing edge trails and hop badges (`Hop 1`, `Hop 2`), while unvisited nodes gracefully dim.
  2. **Minimalist Crawl HUD**:
     ```
     ┌────────────────────────────────────────────────────────────────────────┐
     │ 🧭 Crawl: "Shortest Paths" │ Hop 3 · 12 Nodes │ [🎛 Filters] [📄 Dossier] [✕] │
     └────────────────────────────────────────────────────────────────────────┘
     ```
     - Floating header dock: 480px width, centered at top.
     - Query name + summary stats.
     - `[🎛 Filters]` button: toggles traversal filters (Min similarity, hops).
     - `[📄 Dossier]` button: opens the **Crawl Dossier Sheet**.
     - `[✕]` Exit button: cleanly returns to idle canvas.
  3. **Dedicated "Crawl Dossier" Drawer**:
     - Moves all heavy analytics off the canvas into a dedicated right drawer:
       - Executive Summary of the traversal path.
       - Step-by-step topic narrative trajectory.
       - Exploration breakdown (Digs vs Links vs Jumps analysis).
       - Comparative hypothesis convergence / divergence breakdown.
       - One-click export options: Download as Markdown Report, Download as JSON, or Copy to Clipboard.

---

### 4.6 Reading Experience: Split-Screen Focused Reader

#### Before vs. After Specification

- **Before**: Clicking a citation or "Open Document" opens a giant full-screen modal ([`source-viewer-dialog.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/dialogs/source-viewer-dialog.tsx)) that completely covers the canvas and results sidebar.
- **After**:
  - Implement a **Split-Screen Focused Reader**:
    - When a citation is clicked in the Results Sidebar or Topic Inspector, the workspace smoothly transitions into a 50/50 split view:
      - Left 50%: Canvas camera focused on the target node.
      - Right 50%: Document Viewer displaying the PDF / Markdown document with yellow context span and orange exact match highlights.
    - A maximize button `[⤢]` is available if the user desires a full-screen reading mode.
    - Closing the reader smoothly returns the canvas to 100% width.

---

## 5. Detailed Component Refactoring Specifications

### 5.1 Refactoring Matrix

| Component | Current File | Primary Anti-Pattern | Proposed Refactored Architecture | Target Code Reduction |
| :--- | :--- | :--- | :--- | :--- |
| **Topbar** | [`header.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/overlays/header.tsx) | 3 islands, layout shifts, native `<select>`, 4 flags, limits gauge | Single bar with Breadcrumb Popover + Right Avatar Hub | 911 lines $\to$ ~350 lines |
| **Search Dock** | [`search-bar.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/overlays/search-bar.tsx) | 5 sliders, hover menu, 3 chip categories, hypothesis cycler | 560px Command Pill + Slide-Over Crawl Studio Flyout | 1,657 lines $\to$ ~500 lines |
| **Subject Node** | [`subject-node.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/canvas/subject-node.tsx) | In-place expansion into 500px card with editors and uploaders | Fixed 220x84px Semantic Node Token + Topic Inspector Sheet | 788 lines $\to$ ~220 lines |
| **Crawl UI** | [`crawl-results-canvas.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/canvas/crawl-results-canvas.tsx) | Duplicate canvas, 14 header pills, cockpit density | Compact Crawl HUD + Dedicated Crawl Dossier Drawer | 2,940 lines $\to$ ~800 lines |
| **Inspector** | *New / Unified* | Redundant `NodeDetailModal`, `NodeEditorDialog`, and in-place cards | Unified `TopicInspectorSheet` with tabs: Overview, Docs, Hits | Replaces 3 disparate dialogs |

---

## 6. Actionable Phased Implementation Roadmap

To execute this architectural transformation safely without breaking existing features or user data, implementation is divided into four iterative phases:

### Phase 1: High-Impact Visual Decluttering (Quick Wins)
- **Scope**: Topbar, Search Bar surface, and Badge Consolidation.
- **Tasks**:
  1. Replace native HTML `<select>` in [`header.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/overlays/header.tsx) with a clean `DropdownMenu` breadcrumb component.
  2. Move Gauge limits and Language options from topbar icons into the User Profile dropdown menu.
  3. Consolidate graph status badges into a single compact metadata tooltip.
  4. Fix layout shifts by giving the topbar a unified full-width flex container.
  5. Replace hover-triggered search mode menu with a clean segmented toggle button.

### Phase 2: The Dedicated Topic Inspector & Canvas Cleanliness
- **Scope**: Deprecate in-place canvas node expansion and introduce the Topic Inspector Sheet.
- **Tasks**:
  1. Create `TopicInspectorSheet` ([`components/overlays/topic-inspector-sheet.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/overlays/)) using Radix UI `Sheet` / Tailwind.
  2. Migrate description editor, Markdown study notes, attached sources, upload dropzone, and match citations into `TopicInspectorSheet`.
  3. Simplify [`subject-node.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/canvas/subject-node.tsx): remove all form inputs and upload loops; render only the fixed 220px semantic card.
  4. Wire node click event on canvas to open `TopicInspectorSheet`.
  5. Deprecate redundant `NodeDetailModal` and `NodeEditorDialog`.

### Phase 3: Crawl Interface Unification & The "Crawl Studio" Flyout
- **Scope**: Clean up crawl configuration and consolidate the dual-canvas architecture.
- **Tasks**:
  1. Extract crawl sliders (Digs, Links, Jumps, Matches per Step, Direction) from [`search-bar.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/overlays/search-bar.tsx) into a dedicated `CrawlStudioDrawer`.
  2. Simplify the Search Dock to the minimal 560px Command Pill.
  3. Refactor crawl results to render as an overlay layer on the primary `GraphCanvas` rather than replacing the canvas with a second component.
  4. Consolidate the 14-pill header toolbar in [`crawl-results-canvas.tsx`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/components/canvas/crawl-results-canvas.tsx) into the compact 4-button Crawl HUD and dedicated Crawl Dossier sheet.

### Phase 4: Design System & Token Consolidation
- **Scope**: Stylesheet optimization and visual polish.
- **Tasks**:
  1. Prune legacy and redundant classes from [`styles.css`](file:///home/jbed94/Projects/viacarraria/viacarraria-frontend/src/styles.css) (currently 324 KB, 16,700 lines) by transitioning to Tailwind v4 theme variables.
  2. Enforce the 60-30-10 color rule across dark and light themes.
  3. Standardize all dialogs, sheets, and popovers on consistent padding (16px), border radius (8px / 12px), and soft ambient drop shadows.
  4. Remove continuous looping CSS pulse animations from idle state elements.

---

## 7. Conclusion

By shifting Via Carraria from an **"in-place canvas data explosion"** to the **"Semantic Canvas with Dedicated Detail Views"** pattern, the application will achieve:
1. **Immediate Cognitive Relief**: Users can visually parse their knowledge DAGs without overlapping card boxes.
2. **Superior Ergonomics**: Deep work (editing notes, managing uploads, reading citations, tuning crawls) takes place in spacious, dedicated drawers rather than cramped popovers.
3. **High-Tier Aesthetics**: A disciplined editorial palette, consistent typography hierarchy, and purposeful whitespace elevate the product from a crowded prototype to a world-class educational research platform.
