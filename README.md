# WikiMe Web (`wikime-web`)

**Client Web Application & Presentation Edge**

`wikime-web` is the primary web client and human interaction portal for **WikiMe** (E-Brain) — an autonomous, proactive second-brain knowledge management and conversational recall platform. It delivers low-latency conversational recall, rapid multimedia and manual thought capture, side-by-side conflict reconciliation, and dead-letter ingestion triage.

---

## 1. Core Architectural Capabilities

The web client operates as a decoupled presentation layer focused on user ergonomics, responsive layouts, and real-time interaction. Specific frontend frameworks, component systems, and client libraries are kept modular and decoupled from the underlying architectural specifications:

- **Ultra-Low Latency Conversational Recall**: Real-time Server-Sent Events (SSE) token streaming with responsive typography, collapsible source provenance cards, and human-in-the-loop action confirmations.
- **Sub-Second Optimistic Capture**: Fire-and-forget capture omnibar providing instantaneous (< 200ms) user feedback while offloading ingestion asynchronously to the backend.
- **Rich Document & Formula Presentation**: Structural formatting for markdown, mathematical notations (LaTeX equations), and syntax-highlighted code fences with clipboard actions.
- **Interactive Knowledge Reconciliation**: Side-by-side visual diffing and one-click resolution policies for contradictory or overlapping knowledge claims.
- **Dead-Letter Scrape Recovery**: Diagnostic triage dashboard with manual transcript and raw text fallback input to bypass paywalls and anti-bot defenses.
- **Session & Multi-Tenant State Isolation**: In-memory access token lifecycle management, conversational session tracking, and tenant workspace context routing.

> [!NOTE]
> The concrete frontend framework, styling methodology, and client-side state libraries are intentionally decoupled from this specification and will be finalized during initial scaffolding.

---

## 2. Key Features & Components

1. **Conversational Recall View (`ChatStreamView`)**:
   - Real-time Server-Sent Events (SSE) token streaming.
   - Rich typography rendering with syntax highlighting, LaTeX equations, and collapsible citation cards.
   - Interactive confirmation modals for sensitive actions (e.g., memory deletion).
2. **Lean Omnibar (`QuickCaptureBar`)**:
   - Instant capture for web links and manual thoughts.
   - Optimistic fire-and-forget UI feedback (< 200ms toast response).
3. **Reconciliation Drawer (`ConflictResolutionDrawer`)**:
   - Side-by-side diffing of conflicting knowledge claims.
   - One-click resolution policies (`OVERWRITE_OLD`, `STORE_DUPLICATE`, `SYNTHESIZE_MERGE`, `DISCARD_NEW`).
4. **Failure Recovery View (`TriageDashboard`)**:
   - Inspection of dead-letter queue items (`status: "SCRAPE_FAILED"`).
   - Direct text and transcript manual paste to bypass scraping paywalls and bot blocks.
5. **Secure Browser Session Store (`BrowserAuthStore`)**:
   - Short-lived access token storage and automatic refresh handling.
   - Session identifier (`session_id`) tracking for multi-turn conversational recall.
   - Multi-tenant workspace contextual state switching.

---

## 3. Configuration & Ingress Setup

### Prerequisites

- Modern runtime environment (Node.js LTS or equivalent)
- Standard package manager
- Running instance of the **WikiMe API Gateway** (defaults to `http://localhost:8000`)

### Environment Configuration

Configure the following variables in your local environment configuration (e.g., `.env.local`):

```dotenv
# API Gateway ingress URL
NEXT_PUBLIC_API_GATEWAY_URL=http://localhost:8000

# Client App Environment
NEXT_PUBLIC_APP_ENV=development

# Optional client telemetry / debug logging
NEXT_PUBLIC_DEBUG_LOGGING=true
```

---

## 4. Boundary Rules & Architectural Constraints

To preserve modular isolation, scalability, and security across the WikiMe ecosystem, `wikime-web` must strictly abide by the following architectural boundaries:

| Permitted (In-Scope) | Prohibited (Forbidden) |
| :--- | :--- |
| Rendering web client UI views and layouts | Direct connection to PostgreSQL, Redis, or Vector DBs (Qdrant/Pinecone) |
| Dispatching HTTP/SSE requests to the API Gateway (`/api/v1/*`) | Hosting or executing local LLM/embedding inference engines |
| Client-side input sanitization and form validation | Running scraping pipelines (Playwright, Puppeteer, yt-dlp) |
| Storing transient session state & auth tokens in client memory | Bypassing API Gateway security policies, rate limits, or RBAC |
| Optimistic toast feedback on quick capture (< 200ms) | Polling loops or persistent WebSocket sockets for fire-and-forget ingestion |
| Displaying manual paste fallback for failed scrapes | Solving, computing, or automating CAPTCHA / bot challenge bypasses |

> [!IMPORTANT]
> All data read and write operations must route exclusively through the WikiMe API Gateway via HTTP/1.1 or HTTP/2 REST endpoints and Server-Sent Events (SSE). Never import server-side database clients or internal AI modules directly into the web application.

---

## 5. Documentation Navigation

- [Architecture Specification](docs/ARCHITECTURE.md): Deep-dive into `ChatStreamView`, `QuickCaptureBar`, `ConflictResolutionDrawer`, `TriageDashboard`, and `BrowserAuthStore`.
- [API Contracts & Schemas](docs/API_CONTRACTS.md): Detailed DTO schemas (`SharePayloadDTO`, `ManualNoteDTO`, `ChatQueryDTO`), REST endpoints, and SSE stream protocols.
- [Phase 07 Implementation Roadmap](docs/ROADMAP.md): Step-by-step development checklist extracted from Phase 07 (`STEP_07_01` to `STEP_07_07`).
- [Service Progress Tracker](docs/PROGRESS.md): Active task checklists and progress status for `wikime-web`.
