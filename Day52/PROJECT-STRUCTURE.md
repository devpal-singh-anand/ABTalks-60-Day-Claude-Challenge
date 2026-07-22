# ScholarForge — Project Structure (V1.0)

**Source of truth:** ScholarForge V1.0 Product Vision + PRD + Software Architecture Document §4 + Implementation Blueprint
**Author role:** Principal Software Architect
**Status:** Implementation-ready folder structure — gates Day 3 development
**Scope discipline:** This structure is a 1:1 reflection of the approved Architecture Document §4 folder layout, mapped onto the Next.js + Express + TypeScript stack locked in ARCHITECTURE.md §2. No new top-level folder is introduced that isn't traceable to an architectural layer or an explicit implementation need.

---

## 1. Repository Shape (Monorepo)

ScholarForge V1 is a **pnpm workspace monorepo** with three packages: the Next.js frontend, the Express backend, and a shared types package. The shared package prevents the single most common solo-dev failure mode — circular dependencies and type drift between frontend and backend — by giving both a common, dependency-free vocabulary of data objects and interfaces.

```
scholarforge/
│
├── apps/
│   ├── web/                          # Next.js frontend (Presentation Layer)
│   └── api/                          # Express backend (Application + AI + Formatting + Persistence + Export + Config Layers)
│
├── packages/
│   └── shared/                       # Cross-cutting: data models, zod schemas, I* interfaces
│
├── tools/                            # Dev-only scripts (debug harnesses, fixture generators)
│
├── docs/                             # All planning + architecture documentation
│   ├── product_vision.md
│   ├── prd.md
│   ├── architecture.md               # The approved Software Architecture Document
│   ├── implementation_blueprint.md   # The approved Day 2-10 blueprint
│   ├── installation_guide.md
│   ├── demo_script.md
│   ├── acceptance_checklist_results.md
│   └── adr/                          # Architecture Decision Records
│       ├── ADR-001-deterministic-formatting.md
│       ├── ADR-002-data-driven-templates.md
│       ├── ADR-003-model-backend-tuning.md
│       ├── ADR-004-single-process-monolith.md
│       ├── ADR-005-section-by-section-generation.md
│       ├── ADR-006-no-database.md
│       ├── ADR-007-two-mode-deployment.md
│       └── ADR-008-stateless-backend.md
│
├── .github/
│   └── workflows/
│       └── ci.yml                    # Lint + type-check + unit tests on every push
│
├── .gitignore
├── pnpm-workspace.yaml
├── package.json                      # Root — workspace scripts (dev, build, test, lint)
├── tsconfig.base.json                # Shared TS config — strict mode
├── README.md
├── CHANGELOG.md
└── LICENSE
```

### 1.1 Why a Monorepo (Not Separate Repos)

| Question | Answer |
|---|---|
| Are frontend and backend deployed together? | No — frontend to Vercel, backend runs locally. |
| Do they share types? | **Yes, heavily.** Every API request/response body is a `shared/` type. Separate repos would require publishing the shared package or duplicating types — both worse than a monorepo. |
| Is there a team that benefits from repo-level access control? | No — solo dev. |
| Is the build pipeline different per app? | Yes, but pnpm workspaces handle per-package scripts naturally. |

**Decision:** Monorepo with `apps/web`, `apps/api`, `packages/shared`. The shared package is consumed by both apps via workspace path mapping, never published to a registry in V1.

---

## 2. `apps/web/` — Next.js Frontend (Presentation Layer)

```
apps/web/
│
├── src/
│   ├── app/                          # Next.js App Router
│   │   ├── layout.tsx                # Root layout — providers, theme
│   │   ├── page.tsx                  # Home / project picker page
│   │   ├── project/
│   │   │   └── [projectId]/
│   │   │       ├── layout.tsx        # Project workspace layout (sidebar nav)
│   │   │       ├── page.tsx          # Notes editor (default view)
│   │   │       ├── draft/page.tsx    # Draft view
│   │   │       ├── review/page.tsx   # Reviewer panel
│   │   │       └── export/page.tsx   # Export view
│   │   ├── settings/
│   │   │   └── page.tsx              # Model config, voice profiles
│   │   └── globals.css               # Tailwind directives + global styles
│   │
│   ├── components/
│   │   ├── ui/                       # shadcn/ui components (copy-in, not imported)
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── input.tsx
│   │   │   ├── textarea.tsx
│   │   │   ├── select.tsx
│   │   │   └── ...
│   │   ├── notes/                    # Notes editor components
│   │   │   ├── NotesEditor.tsx
│   │   │   ├── SectionEditor.tsx
│   │   │   ├── BulletList.tsx
│   │   │   ├── SourceRefEditor.tsx
│   │   │   └── ValidationIssues.tsx
│   │   ├── draft/                    # Draft view components
│   │   │   ├── DraftView.tsx
│   │   │   ├── DraftSection.tsx
│   │   │   └── GenerationProgress.tsx
│   │   ├── review/                   # Reviewer panel components
│   │   │   ├── ReviewerPanel.tsx
│   │   │   └── FeedbackCard.tsx
│   │   ├── template/                 # Template picker components
│   │   │   ├── TemplatePicker.tsx
│   │   │   └── TemplateCard.tsx
│   │   ├── export/                   # Export components
│   │   │   ├── ExportDialog.tsx
│   │   │   └── ExportSuccess.tsx
│   │   ├── session/                  # Save/load UI
│   │   │   ├── SaveDialog.tsx
│   │   │   ├── OpenDialog.tsx
│   │   │   └── AutosaveIndicator.tsx
│   │   └── shared/                   # Cross-view shared components
│   │       ├── Header.tsx
│   │       ├── Sidebar.tsx
│   │       └── ErrorBoundary.tsx
│   │
│   ├── lib/                          # Frontend utilities (no business logic)
│   │   ├── api-client.ts             # Typed fetch wrapper for the Express backend
│   │   ├── utils.ts                  # cn() class merger, formatters
│   │   └── constants.ts              # API base URL, route paths
│   │
│   ├── hooks/                        # React hooks
│   │   ├── useGenerateDraft.ts       # TanStack Query mutation for /generate/draft
│   │   ├── useGenerateDraftStream.ts # SSE hook for /generate/draft/stream
│   │   ├── useApplyFormatting.ts     # Mutation for /format/apply
│   │   ├── useExportDocx.ts          # Mutation for /export/docx
│   │   ├── useSession.ts             # Zustand store hook
│   │   └── useAutosave.ts            # Debounced autosave trigger
│   │
│   ├── stores/                       # Zustand stores
│   │   ├── session-store.ts          # Current SessionState (notes, draft, etc.)
│   │   └── ui-store.ts               # UI flags (active view, modals open)
│   │
│   └── types/                        # Frontend-specific types (re-exports from shared/)
│       └── index.ts
│
├── public/                           # Static assets
│   └── favicon.ico
│
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json                     # Extends ../../tsconfig.base.json
├── components.json                   # shadcn/ui config
└── package.json
```

### 2.1 Folder Responsibilities

| Folder | Responsibility | Why It Exists |
|---|---|---|
| `src/app/` | Next.js App Router pages and layouts | Next.js convention — file-based routing. Each route = one view in the UI wireframes. |
| `src/components/ui/` | shadcn/ui primitives | Copy-in components (per shadcn philosophy), not an imported library. Survives upstream breaking changes. |
| `src/components/{notes,draft,review,template,export,session}/` | Feature-grouped components | One folder per major UI feature, matching the views in UI-WIREFRAMES.md. Prevents a flat `components/` folder with 50 files. |
| `src/components/shared/` | Cross-view shared components | Header, sidebar, error boundary — used by every view. |
| `src/lib/` | Pure utilities, no business logic | API client, class mergers, constants. No React, no Zustand. |
| `src/hooks/` | Custom React hooks | Encapsulate TanStack Query mutations and SSE handling. Reusable across views. |
| `src/stores/` | Zustand stores | Client state — `session-store` holds the current `SessionState`, `ui-store` holds UI flags. |
| `src/types/` | Frontend type re-exports | Single import surface for shared types. Frontend code imports from `@scholarforge/shared` via this folder. |

### 2.2 Routing → View Mapping

| Route | View | Wireframe |
|---|---|---|
| `/` | Home / project picker | UI-WIREFRAMES.md §3.1 |
| `/project/[id]` | Notes editor (default workspace view) | UI-WIREFRAMES.md §3.2 |
| `/project/[id]/draft` | Draft view | UI-WIREFRAMES.md §3.3 |
| `/project/[id]/review` | Reviewer panel | UI-WIREFRAMES.md §3.4 |
| `/project/[id]/export` | Export view | UI-WIREFRAMES.md §3.5 |
| `/settings` | Model config, voice profiles | UI-WIREFRAMES.md §3.6 |

---

## 3. `apps/api/` — Express Backend

```
apps/api/
│
├── src/
│   ├── server.ts                     # Express app entry — middleware, route mounting, startup
│   │
│   ├── routes/                       # HTTP route handlers (thin — delegate to workflows)
│   │   ├── health.routes.ts          # GET /health
│   │   ├── config.routes.ts          # GET /config/*
│   │   ├── notes.routes.ts           # POST /notes/validate
│   │   ├── generate.routes.ts        # POST /generate/draft, /generate/draft/stream, /generate/review
│   │   ├── format.routes.ts          # POST /format/apply, GET /format/templates/:id
│   │   ├── export.routes.ts          # POST /export/docx
│   │   └── session.routes.ts         # POST /session/save, /session/load, GET /session/recent
│   │
│   ├── middleware/
│   │   ├── cors.ts                   # CORS config (Vercel + localhost origins)
│   │   ├── error-handler.ts          # Centralized error → RFC 7807 response
│   │   └── request-logger.ts         # Dev-only request logging
│   │
│   ├── application/                  # Application Layer — use-case orchestration
│   │   ├── workflows/
│   │   │   ├── GenerationWorkflow.ts # generateDraft(notes, voice, modelClient): Draft
│   │   │   ├── ReviewWorkflow.ts     # runReview(draft, config): ReviewResult
│   │   │   ├── ExportWorkflow.ts     # exportDocx(formattedDocument, path): ExportResult
│   │   │   └── SessionWorkflow.ts    # saveSession(state, path), loadSession(path)
│   │   └── validation/
│   │       └── NotesValidator.ts     # validate(doc): ValidationResult
│   │
│   ├── ai/                           # AI Layer
│   │   ├── prompt/
│   │   │   ├── PromptBuilder.ts      # build(section, voice, template): Prompt
│   │   │   └── SectionPromptTemplates/  # Data files (JSON), one per section type
│   │   │       ├── introduction.json
│   │   │       ├── literature_review.json
│   │   │       ├── methodology.json
│   │   │       ├── results.json
│   │   │       ├── discussion.json
│   │   │       └── conclusion.json
│   │   ├── client/
│   │   │   ├── IModelClient.ts        # Interface
│   │   │   ├── OllamaClient.ts        # Implementation (localhost:11434)
│   │   │   ├── LlamaCppClient.ts      # Implementation (local process)
│   │   │   └── ModelClientFactory.ts  # Config-driven selector
│   │   ├── cleaning/
│   │   │   └── OutputCleaner.ts       # clean(rawText): CleanedText
│   │   ├── voice/
│   │   │   └── VoiceProfile.ts        # VoiceProfile type + loader
│   │   └── reviewer/
│   │       ├── ReviewerEngine.ts      # review(draft): ReviewerFeedback[]
│   │       └── ReviewerPromptTemplates/
│   │           ├── clarity.json
│   │           ├── argument.json
│   │           ├── evidence.json
│   │           ├── style.json
│   │           └── structure.json
│   │
│   ├── formatting/                   # Formatting Layer (Deterministic)
│   │   ├── engine/
│   │   │   ├── ITemplateEngine.ts     # Interface
│   │   │   ├── TemplateEngine.ts      # Generic engine
│   │   │   ├── LayoutResolver.ts      # Data-driven layout resolution
│   │   │   ├── CitationFormatter.ts   # Strategy-based citation formatting
│   │   │   └── TemplateRegistry.ts    # Loads + caches TemplateDefinition files
│   │   └── templates/                 # DATA, not code — JSON files
│   │       ├── ieee_us_letter.json
│   │       ├── ieee_a4.json
│   │       ├── apa_6.json
│   │       ├── apa_7.json
│   │       ├── mla_8.json
│   │       ├── mla_9.json
│   │       ├── chicago_17.json
│   │       └── chicago_18.json
│   │
│   ├── persistence/                  # Persistence Layer
│   │   ├── ISessionStore.ts          # Interface
│   │   ├── SessionStore.ts           # Atomic-write implementation
│   │   ├── AutosaveScheduler.ts      # (Used frontend-side, but logic lives here)
│   │   └── schema/
│   │       └── session_schema_v1.json # zod schema for SessionState validation
│   │
│   ├── export/                       # Export Layer
│   │   ├── IExporter.ts              # Interface (PDF in V2 sits here too)
│   │   ├── DocxExporter.ts           # Orchestrates DOCX export
│   │   ├── DocxRenderer.ts           # FormattedDocument → OOXML mapping
│   │   └── StyleMapper.ts            # StyleSheet → DOCX style definitions
│   │
│   ├── config/                       # Configuration Layer
│   │   ├── ConfigurationService.ts   # Loads + validates all config at startup
│   │   ├── model.config.json         # Backend, model, quantization, context, timeout
│   │   ├── voice_profile.default.json
│   │   ├── reviewer.config.json
│   │   └── export.config.json        # Default output path, author metadata
│   │
│   └── lib/                          # Backend utilities
│       ├── atomic-write.ts           # temp-file-then-rename helper
│       ├── path-utils.ts             # Path validation (absolute, extension checks)
│       └── logger.ts                 # Structured logging (console in V1)
│
├── tests/
│   ├── unit/
│   │   ├── formatting/               # No model required — fully deterministic
│   │   │   ├── TemplateEngine.test.ts
│   │   │   ├── LayoutResolver.test.ts
│   │   │   └── CitationFormatter.test.ts
│   │   ├── ai/
│   │   │   ├── OutputCleaner.test.ts
│   │   │   └── PromptBuilder.test.ts
│   │   ├── persistence/
│   │   │   ├── SessionStore.test.ts
│   │   │   └── CorruptionRecovery.test.ts
│   │   └── validation/
│   │       └── NotesValidator.test.ts
│   ├── integration/
│   │   ├── GenerationWorkflow.test.ts # Mocked IModelClient
│   │   ├── end_to_end_ieee_pipeline.test.ts
│   │   └── ReviewerEngine.test.ts
│   └── fixtures/
│       └── sample_notes/
│           └── valid_sample.json      # Canonical fixture reused across all days
│
├── tsconfig.json                      # Extends ../../tsconfig.base.json
└── package.json
```

### 3.1 Folder Responsibilities

| Folder | Responsibility | Why It Exists |
|---|---|---|
| `src/routes/` | HTTP route handlers | Thin layer — parse request, call workflow, format response. No business logic. One file per resource group, matching API.md sections. |
| `src/middleware/` | Cross-cutting HTTP concerns | CORS, error handling, logging. Kept separate from routes so routes stay focused. |
| `src/application/` | Use-case orchestration | The Application Layer from Architecture §3. Workflows coordinate AI/Formatting/Persistence/Export layers. No knowledge of HTTP. |
| `src/application/workflows/` | One workflow per use case | `GenerationWorkflow`, `ReviewWorkflow`, `ExportWorkflow`, `SessionWorkflow`. Each is the orchestration logic for one user action. |
| `src/application/validation/` | Input validation | `NotesValidator` — structural checks before generation. |
| `src/ai/` | AI Layer | All language generation. Sub-folders per AI concern (prompt, client, cleaning, voice, reviewer). |
| `src/ai/prompt/SectionPromptTemplates/` | Data files, not code | One JSON per section type. Tunable without redeploying code (ADR-002 pattern). |
| `src/ai/client/` | Model backend adapters | `IModelClient` + Ollama/llama.cpp implementations + factory. ADR-003. |
| `src/formatting/` | Deterministic Formatting Layer | Pure code, no model dependency. Fully unit-testable in isolation. |
| `src/formatting/templates/` | Template data files | JSON, one per standard/edition. Adding a new standard = adding a file, not writing code. ADR-002. |
| `src/persistence/` | Session save/load/autosave | Atomic writes, schema validation, corruption recovery. |
| `src/export/` | DOCX export | `DocxRenderer` maps `FormattedDocument` → OOXML via the `docx` npm library. |
| `src/config/` | Configuration Layer | All tunables. Loaded once at startup by `ConfigurationService`. No other module reads a config file directly. |
| `src/lib/` | Backend utilities | Atomic write helper, path validation, logger. Pure functions, no business logic. |
| `tests/unit/` | Unit tests | One test file per source file. Fast (no model, no I/O for formatting tests). |
| `tests/integration/` | Integration tests | Multi-layer tests with mocked model. `end_to_end_ieee_pipeline` is the regression test for the full Notes→DOCX flow. |
| `tests/fixtures/` | Test fixtures | `valid_sample.json` is the canonical fixture reused across every day's tests. |

---

## 4. `packages/shared/` — Cross-Cutting Types & Schemas

```
packages/shared/
│
├── src/
│   ├── models/                       # Data models (TypeScript interfaces)
│   │   ├── NotesDocument.ts          # NotesDocument, NoteSection, SourceRef, SectionType
│   │   ├── Draft.ts                  # Draft, DraftSection, ModelConfigSnapshot
│   │   ├── FormattedDocument.ts      # FormattedDocument, FormattedSection, StyleSheet, FormattedReference
│   │   ├── ReviewerFeedback.ts       # ReviewerFeedback, ReviewResult
│   │   ├── SessionState.ts           # SessionState, VoiceProfile
│   │   └── Config.ts                 # ModelConfig, ReviewerConfig, TemplateSelection
│   │
│   ├── schemas/                      # zod schemas (runtime validation)
│   │   ├── notes.schema.ts
│   │   ├── draft.schema.ts
│   │   ├── formatted-document.schema.ts
│   │   ├── reviewer-feedback.schema.ts
│   │   ├── session-state.schema.ts
│   │   └── config.schema.ts
│   │
│   ├── interfaces/                   # I* contracts (Ports)
│   │   ├── IModelClient.ts
│   │   ├── ITemplateEngine.ts
│   │   ├── IExporter.ts
│   │   ├── ISessionStore.ts
│   │   └── IConfigurationService.ts
│   │
│   ├── types/                        # Shared utility types
│   │   ├── ValidationResult.ts
│   │   ├── ExportResult.ts
│   │   └── ApiError.ts
│   │
│   └── index.ts                      # Barrel export — single import surface
│
├── tsconfig.json
└── package.json                      # name: "@scholarforge/shared"
```

### 4.1 Why a Shared Package

| Problem | Solution |
|---|---|
| Frontend and backend need the same types for API request/response bodies | Shared package — both import from `@scholarforge/shared` |
| zod schemas must match TypeScript types | Co-located in shared package — `notes.schema.ts` validates `NotesDocument` from `notes.ts` |
| Interface contracts (`IModelClient`, etc.) must be stable across consumers | Defined once in shared, implemented in backend, consumed by workflows |
| Avoid circular dependencies between `apps/web` and `apps/api` | Neither depends on the other — both depend on `packages/shared` |

### 4.2 Import Pattern

```typescript
// In apps/web or apps/api:
import { NotesDocument, NotesDocumentSchema, IModelClient } from '@scholarforge/shared';
```

The shared package is **buildless** in V1 — consumed as raw TypeScript via pnpm workspace path mapping. This avoids a build step and keeps type-checking across the monorepo unified.

---

## 5. `tools/` — Dev-Only Scripts

```
tools/
│
├── debug-model-harness.ts            # Day 3: send a fixed prompt to the model, log response + latency + VRAM
├── generate-fixture.ts               # Regenerate the canonical sample_notes fixture
├── validate-templates.ts             # Validate all formatting/templates/*.json against the schema
└── benchmark-generation.ts           # Day 10: stress-test generation latency against NFR-5
```

### 5.1 Why a Separate `tools/` Folder

| Concern | Reasoning |
|---|---|
| Separation from shipped code | Debug harnesses must never ship to production. Keeping them in `tools/` makes this explicit and audit-able. |
| Different dependencies | Tools may use dev-only dependencies (benchmarking libs, etc.) that shouldn't be in the main app's `package.json`. |
| Clear intent | A reviewer (or the dev returning after a break) immediately sees these are utilities, not app code. |

---

## 6. `docs/` — Documentation

```
docs/
│
├── product_vision.md                 # The approved Vision (source of truth #1)
├── prd.md                            # The approved PRD (source of truth #2)
├── architecture.md                   # The approved Architecture Document (source of truth #3)
├── implementation_blueprint.md       # The approved Blueprint (source of truth #4)
├── installation_guide.md             # Step-by-step setup for a new machine
├── demo_script.md                    # The exact rehearsed demo flow
├── acceptance_checklist_results.md   # Filled-in PRD §10 checklist with pass/fail evidence
└── adr/                              # Architecture Decision Records
    ├── ADR-001-deterministic-formatting.md
    ├── ADR-002-data-driven-templates.md
    ├── ADR-003-model-backend-tuning.md
    ├── ADR-004-single-process-monolith.md
    ├── ADR-005-section-by-section-generation.md
    ├── ADR-006-no-database.md
    ├── ADR-007-two-mode-deployment.md
    └── ADR-008-stateless-backend.md
```

### 6.1 Documentation Organization Rationale

Every planning artifact that governed this build (Vision, PRD, Architecture, Blueprint) ships **inside the repository** under `docs/`, not just referenced externally. The repository itself is a complete, self-contained record of why the system is shaped the way it is. This matters both for a solo developer returning to this project later and for anyone evaluating the project's engineering discipline.

ADRs are one file per decision, named `ADR-NNN-kebab-case-title.md`. Each ADR follows the format: Context → Decision → Reason → Alternatives Considered → Trade-off.

---

## 7. Dependency Boundary Enforcement

### 7.1 Allowed Dependencies

| Source | May Import From |
|---|---|
| `apps/web/**` | `packages/shared`, `apps/web/**` (internal), external UI libs |
| `apps/api/routes/**` | `apps/api/application/**`, `packages/shared` |
| `apps/api/application/**` | `apps/api/ai/**`, `apps/api/formatting/**`, `apps/api/persistence/**`, `apps/api/export/**`, `apps/api/config/**`, `packages/shared` |
| `apps/api/ai/**` | `packages/shared` (NOT `formatting/`, NOT `export/`, NOT `persistence/`) |
| `apps/api/formatting/**` | `packages/shared` (NOT `ai/`, NOT `export/`, NOT `persistence/`) |
| `apps/api/persistence/**` | `packages/shared` (NOT `ai/`, NOT `formatting/`) |
| `apps/api/export/**` | `packages/shared` (NOT `ai/`, NOT `formatting/` — only consumes `FormattedDocument`) |
| `apps/api/config/**` | `packages/shared` |
| `packages/shared/**` | Nothing (pure types + schemas) |

### 7.2 Enforcement

These boundaries are enforced by **ESLint `no-restricted-imports` rule** configured in `tsconfig.base.json`:

```javascript
// eslint config snippet
'no-restricted-imports': ['error', {
  patterns: [
    // ai/ cannot import from formatting/, export/, persistence/
    { group: ['../formatting/*', '../export/*', '../persistence/*'], message: 'AI Layer must not depend on Formatting/Export/Persistence layers.' }
  ]
}]
```

This makes violations a **compile-time error**, not a code-review convention. The boundaries can't erode under time pressure.

---

## 8. Build & Run Scripts

Root `package.json` scripts:

```json
{
  "scripts": {
    "dev": "pnpm --filter @scholarforge/web dev & pnpm --filter @scholarforge/api dev",
    "dev:web": "pnpm --filter @scholarforge/web dev",
    "dev:api": "pnpm --filter @scholarforge/api dev",
    "build": "pnpm --filter @scholarforge/web build && pnpm --filter @scholarforge/api build",
    "test": "pnpm --filter @scholarforge/api test",
    "lint": "pnpm -r lint",
    "type-check": "pnpm -r type-check",
    "format": "prettier --write ."
  }
}
```

| Script | Purpose |
|---|---|
| `pnpm dev` | Run both frontend (:3000) and backend (:3001) in parallel for local development |
| `pnpm dev:web` / `pnpm dev:api` | Run only one side (useful when the other is already running) |
| `pnpm build` | Production build for both apps |
| `pnpm test` | Run the backend test suite (where all business logic tests live) |
| `pnpm lint` | Lint all packages |
| `pnpm type-check` | TypeScript type-checking across the monorepo |
| `pnpm format` | Prettier formatting |

---

## 9. .gitignore Essentials

```gitignore
# Dependencies
node_modules/

# Build outputs
apps/web/.next/
apps/api/dist/

# Local config (user-specific overrides — should not be committed)
apps/api/src/config/*.local.json

# Session files (never commit user project data)
*.scholarforge
*.scholarforge.tmp
*.scholarforge.tmp.bak

# Exported documents
*.docx

# Model weights (if stored locally for some reason — should be in Ollama's dir)
*.gguf
*.bin

# OS files
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/

# Env
.env
.env.local
```

---

## 10. Structure → Architecture Layer Mapping

| Folder | Architecture Layer | Source Doc Reference |
|---|---|---|
| `apps/web/` | Presentation Layer | Architecture §3.1 |
| `apps/api/routes/` + `apps/api/middleware/` | HTTP boundary (part of Presentation for the backend) | API.md §1 |
| `apps/api/application/` | Application Layer | Architecture §3.2 |
| `apps/api/ai/` | AI Layer | Architecture §3.3 |
| `apps/api/formatting/` | Formatting Layer | Architecture §3.4 |
| `apps/api/persistence/` | Persistence Layer | Architecture §3.5 |
| `apps/api/export/` | Export Layer | Architecture §3.6 |
| `apps/api/config/` | Configuration Layer | Architecture §3.7 |
| `packages/shared/` | Shared Kernel (data models + interfaces) | Architecture §4 (`shared/`) |
| `tools/` | Dev tooling (not an architectural layer) | Blueprint Day 3 |
| `docs/` | Documentation | Architecture §4 (`docs/`) |

Every top-level folder maps to exactly one architectural layer (or to cross-cutting concerns like `shared/`, `tools/`, `docs/`). No folder exists "just because" — each has a documented purpose and a traceable source.

---

## 11. References

| Document | Section |
|---|---|
| Software Architecture Document | §4 (Folder Structure — original spec) |
| Implementation Blueprint | GitHub Structure section (end of blueprint) |
| This document | `Day52/PROJECT-STRUCTURE.md` |
| ARCHITECTURE.md | §2 (Final tech stack), §3 (Component diagram) |
| API.md | §8 (Endpoint summary — maps to route files) |
| SCHEMA.md | §3 (Data models — define the shared package contents) |
| UI-WIREFRAMES.md | §3 (View → route mapping) |
