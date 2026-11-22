# WikiMe Web (`wikime-web`)

**Client Web Application & Presentation Edge**

`wikime-web` is the primary web client and human interaction portal for **WikiMe** (E-Brain) — an autonomous, proactive second-brain knowledge management and conversational recall platform. Built with Next.js 15 and modern frontend primitives, it delivers low-latency conversational recall, rapid multimedia and manual thought capture, side-by-side conflict reconciliation, and dead-letter ingestion triage.

---

## 1. Tech Stack

- **Framework**: [Next.js 15+](https://nextjs.org/) (App Router, React Server Components & Client Components)
- **Language**: [TypeScript](https://www.typescriptlang.org/) (Strict Mode)
- **Styling**: [Tailwind CSS](https://tailwindcss.com/) with `@tailwindcss/typography`
- **UI Primitives**: [Radix UI](https://www.radix-ui.com/) via [Shadcn UI](https://ui.shadcn.com/)
- **Icons**: [Lucide React](https://lucide.dev/)
- **State Management**: [Zustand](https://github.com/pmndrs/zustand) (Client auth, active session ID, tenant workspace context)
- **Server State & Caching**: [TanStack Query](https://tanstack.com/query) (React Query v5)
- **Rich Content & Math Rendering**:
  - `react-markdown` + `remark-gfm` (Markdown tables, lists, formatting)
  - `remark-math` + `rehype-katex` (LaTeX mathematical equations)
  - `shiki` / `rehype-pretty-code` (Syntax-highlighted code fences)
- **Notifications & Feedback**: [Sonner](https://sonner.emilkowal.ski/) (Sub-second optimistic toasts)

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
   - Short-lived JWT storage and automatic token refresh handling.
   - Session identifier (`session_id`) tracking for multi-turn conversational recall.
   - Multi-tenant workspace contextual state switching.

---

## 3. Quickstart Guide

### Prerequisites

- **Node.js**: v20.x LTS or higher
- **Package Manager**: `pnpm` (recommended) or `npm` / `yarn`
- **WikiMe API Gateway**: Running backend instance (defaults to `http://localhost:8000`)

### Installation & Environment

1. Clone or navigate into the web workspace:
   ```bash
   cd services/wikime-web
   ```

2. Create a local environment configuration file:
   ```bash
   cp .env.example .env.local
   ```
   Configure the following variables in `.env.local`:
   ```dotenv
   # API Gateway connection
   NEXT_PUBLIC_API_GATEWAY_URL=http://localhost:8000
   
   # App Environment
   NEXT_PUBLIC_APP_ENV=development
   
   # Optional telemetry / debugging
   NEXT_PUBLIC_DEBUG_LOGGING=true
   ```

3. Install project dependencies:
   ```bash
   pnpm install
   ```

### Running Locally

Start the local development server with Turbopack:
```bash
pnpm dev
```
Open [http://localhost:3000](http://localhost:3000) in your browser to access the web application.

### Building for Production

Compile and optimize the Next.js production build:
```bash
pnpm build
pnpm start
```

---

## 4. Boundary Rules & Architectural Constraints

To preserve modular isolation, scalability, and security across the WikiMe ecosystem, `wikime-web` must strictly abide by the following architectural boundaries:

| Permitted (In-Scope) | Prohibited (Forbidden) |
| :--- | :--- |
| Rendering UI components and views | Direct connection to PostgreSQL, Redis, or Vector DBs (Qdrant/Pinecone) |
| Dispatching HTTP/SSE requests to the API Gateway (`/api/v1/*`) | Hosting or executing local LLM/embedding inference engines |
| Client-side input sanitization and form validation | Running scraping pipelines (Playwright, Puppeteer, yt-dlp) |
| Storing transient session state & JWT auth tokens | Bypassing API Gateway security policies, rate limits, or RBAC |
| Optimistic toast feedback on quick capture (< 200ms) | Polling loops or persistent WebSocket sockets for fire-and-forget ingestion |

> [!IMPORTANT]
> All data read and write operations must route exclusively through the WikiMe API Gateway via HTTP/1.1 or HTTP/2 REST endpoints and Server-Sent Events (SSE). Never import server-side database clients or internal AI modules directly into the web application.

---

## 5. Documentation Navigation

- [Architecture Specification](docs/ARCHITECTURE.md): Deep-dive into `ChatStreamView`, `QuickCaptureBar`, `ConflictResolutionDrawer`, `TriageDashboard`, and `BrowserAuthStore`.
- [API Contracts & Schemas](docs/API_CONTRACTS.md): Detailed DTO schemas (`SharePayloadDTO`, `ManualNoteDTO`, `ChatQueryDTO`), REST endpoints, and SSE stream protocols.
- [Phase 07 Implementation Roadmap](docs/ROADMAP.md): Step-by-step development checklist extracted from Phase 07 (`STEP_07_01` to `STEP_07_07`).
- [WikiMe Umbrella Repository](../../README.md): Master workspace overview and cross-service architectural topology.
- [WikiMe Master Progress Tracker](../../docs/PROGRESS.md): Master progress tracking across all 11 implementation phases.
