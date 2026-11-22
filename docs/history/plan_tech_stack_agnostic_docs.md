# Implementation Plan: Remove Hardcoded Frontend Tech Stack from Documentation

Remove all hardcoded frontend frameworks and libraries (Next.js 15, Tailwind CSS, Shadcn UI, Radix UI, Zustand, TanStack Query, Sonner, react-markdown, Shiki, KaTeX packages, Turbopack, pnpm) from the documentation, preserving pure architectural design, component responsibilities, data-flow diagrams, and API contracts.

## Phase 1: Progress Tracking & Sprint Alignment
- [x] Check and align `docs/PROGRESS.md` and `SPRINT.md` with tech-agnostic Phase 07 checkpoints

## Phase 2: Refactor `README.md`
- [x] Replace Section 1 ("Tech Stack") with "Architecture & Presentation Capabilities"
- [x] Replace framework-specific Quickstart/build commands with generic web application configuration guidelines
- [x] Remove hardcoded library references from Key Features and Boundary Rules

## Phase 3: Refactor `docs/ARCHITECTURE.md`
- [x] Update High-Level Overview and Mermaid system architecture diagram to be technology-agnostic
- [x] Neutralize component specifications (`ChatStreamView`, `QuickCaptureBar`, `ConflictResolutionDrawer`, `TriageDashboard`, `BrowserAuthStore`) by focusing on functional capabilities, events, and protocols
- [x] Update In-Scope vs Forbidden Boundary Matrix to remove specific framework names

## Phase 4: Refactor `docs/API_CONTRACTS.md`
- [x] Replace framework/library-specific examples in sample payloads and citations with generic architectural content

## Phase 5: Refactor `docs/ROADMAP.md`
- [x] Update Steps 7.1 to 7.7 (`STEP_07_01` - `STEP_07_07`) and Definition of Done to specify functional deliverables instead of specific libraries

## Phase 6: Verification & Final Status Review
- [x] Run full repository search to ensure no hardcoded frontend library/framework references remain in `docs/` or `README.md`
- [x] Check git diff and verify documentation integrity and clarity
