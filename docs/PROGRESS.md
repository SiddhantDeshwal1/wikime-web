# Master Progress Tracker: `wikime-web`

Master task tracking and implementation roadmap for `wikime-web` (Presentation Edge & Web Client).

---

## Active Sprint & Roadmap Tasks

### Phase 07: Web Client App & Presentation Edge (`PHASE_07`)

- [x] **V1_001: Documentation & Architecture Specifications**
  - [x] Author Service Overview & Presentation Capabilities (`README.md`)
  - [x] Author Architecture Specifications & Component Hierarchy (`docs/ARCHITECTURE.md`)
  - [x] Author API Contracts, Protocols & DTO Schemas (`docs/API_CONTRACTS.md`)
  - [x] Author Implementation Roadmap (`docs/ROADMAP.md`)
  - [x] Establish Governance Rules and Git Commit Guardrails (`RULES.md`, `GEMINI.md`, `SPRINT.md`)

- [ ] **V1_002: Project Scaffold & Design System Architecture (`STEP_07_01`)**
  - [ ] Initialize modular web client application structure and strict type environment
  - [ ] Establish design tokens, typography scale, CSS variables, and dark/light theming foundation
  - [ ] Implement core layout shells (sidebar, global navigation, view containers)
  - [ ] Configure client environment variables (`NEXT_PUBLIC_API_GATEWAY_URL`, app environments)

- [ ] **V1_003: QuickCaptureBar & Sub-Second Capture UX (`STEP_07_02`)**
  - [ ] Implement omnibar interface with global keyboard shortcut (`Cmd/Ctrl + K`)
  - [ ] Automatic input classification (URL vs. manual notes)
  - [ ] Optimistic fire-and-forget UI feedback (< 200ms toast response)
  - [ ] Background API Gateway dispatch to `POST /api/v1/ingest` with retry handling

- [ ] **V1_004: Rich Typography, LaTeX Math & Code Syntax Engine (`STEP_07_03`)**
  - [ ] Markdown parsing pipeline supporting CommonMark and GFM (tables, checklists)
  - [ ] Mathematical equation rendering engine (LaTeX inline and display formulas)
  - [ ] Syntax-highlighted code block renderer with line numbers, language tagging, and 1-click copy action
  - [ ] Clean responsive prose styling

- [ ] **V1_005: Conversational Chat View & Real-Time SSE Streaming (`STEP_07_04`)**
  - [ ] Real-time Server-Sent Events (SSE) stream consumer for `POST /api/v1/chat`
  - [ ] Token chunk rendering with autoscroll pinned to bottom
  - [ ] Collapsible citation provenance cards (source URL, title, chunk snippet, similarity score)
  - [ ] Human-in-the-loop tool-call confirmation modal wired to `POST /api/v1/actions/:id/confirm`

- [ ] **V1_006: Dead Reports Triage Dashboard (`STEP_07_05`)**
  - [ ] Dedicated triage view for failed scraping items (`GET /api/v1/dead-reports`)
  - [ ] Diagnostic metadata rendering (HTTP failure codes, retry counts, error traces)
  - [ ] Expandable manual transcript/text paste interface
  - [ ] Direct re-indexing submission to `POST /api/v1/dead-reports/:id/resolve` (Flow A)

- [ ] **V1_007: Conflict Resolution Drawer (`STEP_07_06`)**
  - [ ] Side-by-side reconciliation interface querying `GET /api/v1/conflicts`
  - [ ] Left pane: Existing memory; Right pane: Incoming claim with similarity score and conflict type
  - [ ] 4 distinct resolution actions: `OVERWRITE_OLD`, `STORE_DUPLICATE`, `SYNTHESIZE_MERGE`, `DISCARD_NEW`
  - [ ] Dispatch to `POST /api/v1/conflicts/:id/resolve` with optimistic updates

- [ ] **V1_008: Client Authentication & Session Persistence (`STEP_07_07`)**
  - [ ] Client auth and session store managing short-lived access JWT in memory
  - [ ] Multi-turn conversational `session_id` persistence across browser reloads
  - [ ] Multi-tenant workspace and organization context tracking (`X-Workspace-Id`, `X-Org-Id`)
  - [ ] Request interceptors for automated Bearer token attachment and silent refresh coordination
