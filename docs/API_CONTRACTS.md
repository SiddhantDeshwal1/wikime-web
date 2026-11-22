# API Contracts & Schemas: `wikime-web`

**Client-to-Gateway API Specifications & Data Transfer Objects**

This document specifies the Data Transfer Objects (DTOs), network protocols, Server-Sent Events (SSE) streaming specifications, and REST endpoints consumed by `wikime-web` when communicating with the WikiMe API Gateway.

---

## 1. Global Gateway Conventions

- **Base URL**: Set by `NEXT_PUBLIC_API_GATEWAY_URL` (e.g., `http://localhost:8000`).
- **Authentication**: Bearer token via `Authorization: Bearer <JWT>` header on all protected routes.
- **Content Type**: `application/json` for standard REST payloads, `text/event-stream` for conversational streaming.
- **Context Headers**:
  - `X-Workspace-Id`: UUID of the active workspace.
  - `X-Org-Id`: UUID of the active organization (for multi-tenant workspaces).
  - `X-Request-Id`: Client-generated UUID for end-to-end request tracing.
- **Error Format**: Conforms to [RFC 7807 Problem Details](https://datatracker.ietf.org/doc/html/rfc7807):
  ```json
  {
    "type": "https://wikime.internal/errors/invalid-payload",
    "title": "Invalid Payload",
    "status": 400,
    "detail": "The 'source_url' parameter must be a valid HTTP or HTTPS URI.",
    "instance": "/api/v1/ingest",
    "code": "VALIDATION_FAILED"
  }
  ```

---

## 2. Data Transfer Objects (DTOs)

### 2.1 Ingestion Share Payload (`SharePayloadDTO`)

Used by `QuickCaptureBar` to submit web links (articles, YouTube videos, tweets, social posts) for ingestion.

#### TypeScript Interface
```typescript
export interface SharePayloadDTO {
  user_id: string;
  source_url: string;
  raw_text: null;
  content_type: "url";
  client_timestamp: string; // ISO-8601 string, e.g. "2026-10-11T13:30:00Z"
}
```

#### JSON Sample
```json
{
  "user_id": "usr_98a7f6c2",
  "source_url": "https://www.instagram.com/reel/C8xYz12345/?igsh=NzBmMjdhZWRiYQ==",
  "raw_text": null,
  "content_type": "url",
  "client_timestamp": "2026-10-11T13:30:00Z"
}
```

---

### 2.2 Manual Note Payload (`ManualNoteDTO`)

Used by `QuickCaptureBar` to submit direct textual notes, ideas, personal rules, or architectural preferences.

#### TypeScript Interface
```typescript
export interface ManualNoteDTO {
  user_id: string;
  source_url: null;
  raw_text: string;
  content_type: "manual_note";
  client_timestamp: string; // ISO-8601 string
}
```

#### JSON Sample
```json
{
  "user_id": "usr_98a7f6c2",
  "source_url": null,
  "raw_text": "I prefer using Shadcn UI over standard Tailwind CSS components for clean dashboard architecture.",
  "content_type": "manual_note",
  "client_timestamp": "2026-10-11T13:31:00Z"
}
```

---

### 2.3 Client Chat Request (`ChatQueryDTO`)

Used by `ChatStreamView` to initiate a conversational synthesis or factual recall query.

#### TypeScript Interface
```typescript
export interface ChatQueryDTO {
  user_id: string;
  session_id: string;
  query: string;
  client_timestamp: string; // ISO-8601 string
}
```

#### JSON Sample
```json
{
  "user_id": "usr_98a7f6c2",
  "session_id": "ses_4b82e1",
  "query": "What was the recommended frontend component library I saved recently?",
  "client_timestamp": "2026-10-11T13:35:00Z"
}
```

---

### 2.4 Conflict Resolution DTOs

Used by `ConflictResolutionDrawer` to inspect and resolve overlapping knowledge claims.

#### TypeScript Interfaces
```typescript
export type ConflictResolutionAction =
  | "OVERWRITE_OLD"
  | "STORE_DUPLICATE"
  | "SYNTHESIZE_MERGE"
  | "DISCARD_NEW";

export type ConflictType = "EXACT_OVERLAP" | "HIGH_PARTIAL_OVERLAP" | "FACTUAL_CONTRADICTION";

export interface ConflictItemDTO {
  conflict_id: string;
  conflict_type: ConflictType;
  similarity_score: number; // e.g. 0.98, 0.85
  reason: string;
  detected_at: string;
  existing_record: {
    item_id: string;
    title: string;
    summary: string;
    content: string;
    source_url?: string;
    indexed_at: string;
  };
  incoming_record: {
    item_id: string;
    title: string;
    summary: string;
    content: string;
    source_url?: string;
    submitted_at: string;
  };
}

export interface ConflictResolutionPayloadDTO {
  action: ConflictResolutionAction;
}
```

---

### 2.5 Dead Reports DTOs

Used by `TriageDashboard` to list failed scrapes and manually submit transcript text for recovery.

#### TypeScript Interfaces
```typescript
export interface DeadReportDTO {
  id: string;
  user_id: string;
  source_url: string;
  error_code: string;
  error_message: string;
  retry_count: number;
  failed_at: string;
  status: "SCRAPE_FAILED";
}

export interface ResolveDeadReportPayloadDTO {
  raw_text: string;
  title?: string;
}
```

---

### 2.6 Action Confirmation DTO

Used by `ChatStreamView` when confirming human-in-the-loop tool actions (e.g., destructive memory deletion).

#### TypeScript Interface
```typescript
export interface ActionConfirmationPayloadDTO {
  action_id: string;
  confirmed: boolean;
  notes?: string;
}
```

---

## 3. Endpoints & Protocol Specifications

### 3.1 Content Ingestion

- **Endpoint**: `POST /api/v1/ingest`
- **Headers**:
  - `Authorization: Bearer <JWT>`
  - `Content-Type: application/json`
- **Request Body**: `SharePayloadDTO | ManualNoteDTO`
- **Status Codes & Responses**:
  - `202 Accepted`: Asynchronous job queued for processing.
    ```json
    {
      "job_id": "job_e71b2a8c",
      "status": "queued",
      "message": "Content enqueued for background processing."
    }
    ```
  - `200 OK`: Fast idempotency gate detected duplicate canonical URL already indexed.
    ```json
    {
      "status": "already_indexed",
      "canonical_url": "https://instagram.com/reel/C8xYz12345",
      "message": "Content is already present in your knowledge base."
    }
    ```
  - `400 Bad Request`: Validation failure.
  - `401 Unauthorized`: Missing or invalid Bearer token.

---

### 3.2 Conversational Chat Streaming (SSE)

- **Endpoint**: `POST /api/v1/chat`
- **Headers**:
  - `Authorization: Bearer <JWT>`
  - `Content-Type: application/json`
  - `Accept: text/event-stream`
- **Request Body**: `ChatQueryDTO`
- **Response**: `200 OK` with chunked Server-Sent Events (`Transfer-Encoding: chunked`).

#### Event Stream Protocol

```http
HTTP/1.1 200 OK
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive

event: token
data: {"token": "Based"}

event: token
data: {"token": " on your"}

event: token
data: {"token": " saved notes,"}

event: citation
data: {"item_id": "mem_01", "title": "Frontend Library Preference", "url": null, "similarity": 0.94, "snippet": "I prefer using Shadcn UI over standard Tailwind CSS components for clean dashboard architecture."}

event: token
data: {"token": " you strongly prefer Shadcn UI."}

event: done
data: {"status": "complete", "total_tokens": 42, "finish_reason": "stop"}
```

#### Tool-Call / Confirmation Handshake Stream Sample

When a user instructs the assistant to perform a destructive operation (e.g., *"Forget my notes about React Table"*):

```http
event: token
data: {"token": "I will proceed with removing that record from your memory."}

event: confirm_action
data: {"action_id": "act_991", "tool_name": "delete_memory", "message": "Delete note: 'Configuring React Table'?", "target_item_id": "mem_45"}

event: done
data: {"status": "awaiting_confirmation", "action_id": "act_991"}
```

---

### 3.3 Conflict Management Endpoints

#### List Pending Conflicts
- **Endpoint**: `GET /api/v1/conflicts`
- **Headers**: `Authorization: Bearer <JWT>`
- **Response**: `200 OK`
  ```json
  {
    "conflicts": [
      {
        "conflict_id": "cnf_3301",
        "conflict_type": "HIGH_PARTIAL_OVERLAP",
        "similarity_score": 0.89,
        "reason": "89% semantic match detected with existing note 'Tailwind Setup Guide'",
        "detected_at": "2026-10-11T14:10:00Z",
        "existing_record": {
          "item_id": "mem_88",
          "title": "Tailwind Setup Guide",
          "summary": "Notes on setting up Tailwind CSS with PostCSS.",
          "content": "Install tailwindcss postcss autoprefixer...",
          "indexed_at": "2026-09-01T10:00:00Z"
        },
        "incoming_record": {
          "item_id": "mem_99",
          "title": "Tailwind v4 Setup",
          "summary": "Notes on setting up Tailwind CSS v4 using @tailwindcss/vite.",
          "content": "Install @tailwindcss/vite and configure vite plugin...",
          "submitted_at": "2026-10-11T14:09:50Z"
        }
      }
    ]
  }
  ```

#### Resolve Pending Conflict
- **Endpoint**: `POST /api/v1/conflicts/:id/resolve`
- **Headers**: `Authorization: Bearer <JWT>`, `Content-Type: application/json`
- **Request Body**:
  ```json
  {
    "action": "SYNTHESIZE_MERGE"
  }
  ```
- **Response**: `200 OK`
  ```json
  {
    "status": "resolved",
    "conflict_id": "cnf_3301",
    "action_taken": "SYNTHESIZE_MERGE",
    "resulting_item_id": "mem_88",
    "message": "Both records successfully synthesized into consolidated knowledge entity."
  }
  ```

---

### 3.4 Dead Reports (Failed Scrapes) Endpoints

#### Fetch Dead-Letter Scraping Reports
- **Endpoint**: `GET /api/v1/dead-reports`
- **Headers**: `Authorization: Bearer <JWT>`
- **Response**: `200 OK`
  ```json
  {
    "failed_items": [
      {
        "id": "dlq_501",
        "user_id": "usr_98a7f6c2",
        "source_url": "https://example.com/walled-article",
        "error_code": "HTTP_403_BOT_BLOCK",
        "error_message": "Cloudflare Turnstile challenge intercepted; automated scraping halted.",
        "retry_count": 3,
        "failed_at": "2026-10-11T12:00:00Z",
        "status": "SCRAPE_FAILED"
      }
    ]
  }
  ```

#### Resolve Dead-Letter Report via Manual Paste
- **Endpoint**: `POST /api/v1/dead-reports/:id/resolve`
- **Headers**: `Authorization: Bearer <JWT>`, `Content-Type: application/json`
- **Request Body**:
  ```json
  {
    "raw_text": "Building resilient event-driven architectures requires decoupling ingestion buffers from vector embedding pipelines...",
    "title": "Resilient Event-Driven Architectures"
  }
  ```
- **Response**: `200 OK`
  ```json
  {
    "status": "requeued",
    "job_id": "job_9981",
    "message": "Manual text received. Bypassed scraping and routed to AI Structurer (Flow A)."
  }
  ```

---

### 3.5 Action Confirmation Endpoint

- **Endpoint**: `POST /api/v1/actions/:id/confirm` (or `POST /api/v1/actions/confirm` with `action_id` in body)
- **Headers**: `Authorization: Bearer <JWT>`, `Content-Type: application/json`
- **Request Body**:
  ```json
  {
    "confirmed": true
  }
  ```
- **Response**: `200 OK`
  ```json
  {
    "status": "executed",
    "action_id": "act_991",
    "result": "Memory 'mem_45' successfully forgotten and purged from vector storage."
  }
  ```
