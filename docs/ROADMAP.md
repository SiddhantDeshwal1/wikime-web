# Implementation Roadmap: `wikime-web`

**Phase 07: Client Web App & Presentation Edge Implementation Checklist**

This roadmap details the engineering checklist, technical deliverables, and verification procedures for `wikime-web`, extracted directly from `PHASE_07: Client Web App & Presentation Edge` of the master WikiMe roadmap.

---

## 1. Phase Overview & Prerequisites

- **Master Phase**: `PHASE_07` (Steps `STEP_07_01` through `STEP_07_07`)
- **Primary Objective**: Build a responsive, accessible, sub-second web client delivering rapid fire-and-forget ingestion, real-time conversational streaming with rich typography, conflict reconciliation, and dead-letter scrape triage.
- **Prerequisites**:
  - `PHASE_02`: API Gateway ingress routes and auth middleware.
  - `PHASE_05`: Conversational retrieval & SSE streaming pipeline.
  - `PHASE_06`: Memory management and conflict triage endpoints.
- **Independence Guarantee**: `wikime-web` can be developed, tested, and styled against mock JSON/SSE handlers prior to live gateway deployment.

---

## 2. Actionable Implementation Steps

### Step 7.1 [STEP_07_01]: Project Scaffold & Design System Architecture
- [ ] Initialize modular web client application structure and strict type environment.
- [ ] Establish design tokens, typography scale, CSS variables, and dark/light theming foundation.
- [ ] Implement accessible UI primitives and layout components (Button, Input, Textarea, Dialog, Drawer, Toast, Tooltip, DropdownMenu).
- [ ] Implement responsive root shell with global navigation, collapsible sidebar drawer, and theme toggler.
- [ ] Set up client environment configuration (`.env.local`) pointing to `NEXT_PUBLIC_API_GATEWAY_URL`.

**Acceptance Criteria**:
- Clean responsive layout renders on desktop and mobile viewports with zero console warnings.
- Dark/light theme switching operates smoothly with zero flash of unstyled content (FOUC).

---

### Step 7.2 [STEP_07_02]: QuickCaptureBar & Optimistic Fire-and-Forget UX
- [ ] Create `QuickCaptureBar` omnibar component accessible globally via keyboard shortcut (`Cmd/Ctrl + K` or floating action).
- [ ] Implement automatic URL vs. plain-text classifier.
- [ ] Construct payload DTOs:
  - URLs $\rightarrow$ `SharePayloadDTO` (`content_type: "url"`)
  - Plain text $\rightarrow$ `ManualNoteDTO` (`content_type: "manual_note"`)
- [ ] Implement sub-second optimistic toast (< 200ms feedback): *"Saved! Will be searchable shortly."*
- [ ] Immediately clear input upon submission to enable continuous capturing without waiting for downstream indexing.
- [ ] Wire background HTTP dispatch to `POST /api/v1/ingest` via asynchronous client mutation with retry handling.

**Acceptance Criteria**:
- Omnibar dismisses or resets in under 200ms following Enter/Submit.
- Validated payloads successfully reach `POST /api/v1/ingest` and receive `202 Accepted` or `200 OK`.

---

### Step 7.3 [STEP_07_03]: Markdown, LaTeX Math & Code Syntax Engine
- [ ] Configure markdown parsing engine supporting CommonMark and GFM specifications (tables, task lists, strikethrough).
- [ ] Integrate mathematical formula rendering engine supporting inline ($E = mc^2$) and display block mathematical notation.
- [ ] Configure syntax-highlighted code block rendering with language tagging, line numbers, and a 1-click copy button.
- [ ] Implement clean responsive prose typography styling across light and dark modes.

**Acceptance Criteria**:
- Formatted markdown renders correctly with zero layout shifts.
- Mathematical formulas ($E = mc^2$, matrix notation) render crisply.
- Code blocks highlight correctly across major programming languages (Python, TypeScript, SQL, Bash).

---

### Step 7.4 [STEP_07_04]: Conversational Chat View & SSE Token Streaming (`ChatStreamView`)
- [ ] Implement `ChatStreamView` message container with autoscrolling pinned to bottom during active streaming.
- [ ] Implement real-time Server-Sent Events (SSE) stream consumer handling chunked event transfer.
- [ ] Handle discrete SSE event channels:
  - `event: token`: Incrementally append token chunks to current assistant message.
  - `event: citation`: Collect source provenance records into collapsible footer cards.
  - `event: confirm_action`: Intercept destructive actions (e.g., `delete_memory`) and display `ActionConfirmationModal`.
  - `event: error`: Display inline error banner with retry action.
  - `event: done`: Commit turn to session state and stop stream spinner.
- [ ] Implement collapsible citation cards showing title, canonical URL, similarity score, and excerpt snippet.
- [ ] Implement confirmation modal for tool actions, dispatching approval to `POST /api/v1/actions/:id/confirm`.

**Acceptance Criteria**:
- Time to First Token (TTFT) perceived in < 250ms when streaming from gateway.
- User can interactively confirm or reject memory deletion tool calls directly from chat.

---

### Step 7.5 [STEP_07_05]: Dead Reports Triage Dashboard (`TriageDashboard`)
- [ ] Create `/triage` page route hosting the `TriageDashboard` component.
- [ ] Fetch dead-letter scraping reports via `GET /api/v1/dead-reports` with client data caching.
- [ ] Render tabular view displaying failed URL, HTTP error code (e.g., 403, 429), failure timestamp, and retry attempts.
- [ ] Build expandable manual transcript paste drawer with a multi-line textarea.
- [ ] Wire submit button to `POST /api/v1/dead-reports/:id/resolve` with pasted `raw_text`.
- [ ] Optimistically remove resolved item from the failure list upon successful submission.

**Acceptance Criteria**:
- Failed items are listed clearly with actionable diagnostic information.
- Pasting raw text resolves the failure and transitions the record to the AI Structuring queue.

---

### Step 7.6 [STEP_07_06]: Conflict Resolution Drawer (`ConflictResolutionDrawer`)
- [ ] Create `ConflictResolutionDrawer` slide-over component.
- [ ] Wire conflict badge notification reading from `GET /api/v1/conflicts`.
- [ ] Implement side-by-side comparison layout:
  - **Left pane**: Existing saved memory (title, summary, body, indexed timestamp).
  - **Right pane**: Incoming conflicting note (title, summary, body, similarity score, conflict type).
- [ ] Implement 4 distinct resolution action buttons:
  - `OVERWRITE_OLD`: Purge existing record, commit incoming.
  - `STORE_DUPLICATE`: Preserve both records concurrently.
  - `SYNTHESIZE_MERGE`: Instruct AI Structurer to synthesize both into a single entity.
  - `DISCARD_NEW`: Drop incoming payload, keep existing memory intact.
- [ ] Wire actions to `POST /api/v1/conflicts/:id/resolve` with loading spinners and optimistic reconciliation.

**Acceptance Criteria**:
- Side-by-side diff clearly highlights semantic discrepancies.
- Triggering any of the 4 actions sends the proper payload and updates UI status without refresh.

---

### Step 7.7 [STEP_07_07]: Authentication Store & Session Persistence (`BrowserAuthStore`)
- [ ] Implement centralized `BrowserAuthStore` client session and authentication management with strict type interfaces.
- [ ] Manage short-lived access JWT in memory; coordinate HttpOnly refresh cookies via gateway `/api/v1/auth/refresh`.
- [ ] Persist active `session_id` in browser session storage for conversational continuity across page reloads.
- [ ] Implement workspace and organization context switching (`workspace_id`, `org_id`).
- [ ] Configure client HTTP interceptors to automatically attach `Authorization: Bearer <JWT>` and context headers.

**Acceptance Criteria**:
- Authenticated requests automatically include Bearer tokens and workspace headers.
- Expired tokens seamlessly trigger refresh without disrupting user conversational flow.

---

## 3. Definition of Done (DoD) for Phase 07

1. All 7 steps (`STEP_07_01` through `STEP_07_07`) fully implemented, typed, and checked off.
2. End-to-end integration verified against mock or live API Gateway endpoints (`/api/v1/*`).
3. Zero type-checking compiler errors.
4. Production build passes cleanly with zero linting or client hydration errors.
5. Sub-second fire-and-forget ingestion response verified (< 200ms toast presentation).
