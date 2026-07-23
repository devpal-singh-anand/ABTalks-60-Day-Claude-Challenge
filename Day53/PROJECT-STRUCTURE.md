# ScholarForge — Project Structure (Day 3 Update)

**Source of truth:** Day 52 PROJECT-STRUCTURE.md + Day 3 implementation
**Status:** Updated to reflect actual structure created on Day 3
**Changes from Day 52:** Minor — structure matches the approved design. Updates noted below.

---

## 1. Current Structure (as of Day 3)

```
D:\DevSpace\scholarforge\
│
├── apps/
│   ├── api/                          # Express backend (Port 3001)
│   │   ├── src/
│   │   │   ├── server.ts             # ✅ CREATED Day 3 — Express app entry
│   │   │   ├── routes/               # Empty (Day 4+)
│   │   │   └── middleware/           # Empty (Day 4+)
│   │   ├── package.json              # @scholarforge/api
│   │   ├── tsconfig.json             # TypeScript config
│   │   └── node_modules/             # (gitignored)
│   │
│   └── web/                          # Next.js frontend (Port 3000)
│       ├── src/
│       │   └── app/
│       │       ├── layout.tsx        # Root layout (Next.js default)
│       │       ├── page.tsx          # Home page (Next.js default — to be replaced Day 4)
│       │       └── globals.css       # Tailwind imports
│       ├── public/                   # Static assets
│       ├── package.json              # @scholarforge/web
│       ├── next.config.ts            # Next.js config
│       ├── tsconfig.json             # TypeScript config
│       ├── postcss.config.mjs        # PostCSS config (Tailwind)
│       ├── eslint.config.mjs         # ESLint config
│       └── node_modules/             # (gitignored)
│
├── packages/
│   └── shared/                       # Shared types + schemas
│       ├── src/
│       │   ├── models/               # Empty (Day 4+)
│       │   ├── schemas/              # Empty (Day 4+)
│       │   └── interfaces/           # Empty (Day 4+)
│       └── package.json              # @scholarforge/shared
│
├── package.json                      # Root workspace package
├── pnpm-workspace.yaml               # Workspace config + build approvals
├── pnpm-lock.yaml                    # Lockfile
├── .gitignore                        # Git ignore rules
└── .git/                             # Git repository
```

---

## 2. Changes from Day 52 Design

### 2.1 What Matches the Day 52 Design ✅

| Item | Status |
|---|---|
| Monorepo shape (`apps/`, `packages/`) | ✅ Matches |
| `apps/api/` (Express backend) | ✅ Matches |
| `apps/web/` (Next.js frontend) | ✅ Matches |
| `packages/shared/` (shared types) | ✅ Matches |
| pnpm workspace | ✅ Matches |
| `.gitignore` content | ✅ Matches |

### 2.2 What's Different (Minor Updates)

| Item | Day 52 Design | Day 3 Actual | Reason |
|---|---|---|---|
| `tools/` folder | Listed in Day 52 | Not created yet | Will be created on Day 4 when we build the debug harness |
| `docs/` folder | Listed in Day 52 | Not created yet | Will be created when we add project documentation |
| `pnpm-workspace.yaml` | Only `packages:` field | Added `onlyBuiltDependencies:` field | pnpm v11 security feature requires explicit build-script approval |
| `apps/api/src/` subfolders | Full structure (`application/`, `ai/`, `formatting/`, etc.) | Only `routes/` and `middleware/` created | Other folders created on their respective days (Day 4+) |

### 2.3 What's Intentionally Empty (To Be Filled on Later Days)

| Folder | When It Gets Populated |
|---|---|
| `apps/api/src/application/` | Day 4 (GenerationWorkflow) |
| `apps/api/src/ai/` | Day 3-4 (IModelClient, PromptBuilder) |
| `apps/api/src/formatting/` | Day 5 (TemplateEngine) |
| `apps/api/src/persistence/` | Day 7 (SessionStore) |
| `apps/api/src/export/` | Day 6 (DocxExporter) |
| `apps/api/src/config/` | Day 3-4 (ConfigurationService) |
| `packages/shared/src/models/` | Day 4 (NotesDocument, Draft, etc.) |
| `packages/shared/src/schemas/` | Day 4 (zod schemas) |
| `packages/shared/src/interfaces/` | Day 3-4 (IModelClient, etc.) |
| `apps/web/src/components/` | Day 4+ (UI components) |
| `apps/web/src/hooks/` | Day 4+ (React hooks) |
| `apps/web/src/stores/` | Day 4+ (Zustand stores) |

---

## 3. Folder Responsibilities (Current State)

### 3.1 `apps/api/src/server.ts`

**Purpose:** Express application entry point.
**Status:** ✅ Created and working.
**Contains:**
- Express app initialization
- CORS middleware (allows `localhost:3000` and Vercel origin)
- JSON body parser
- `GET /api/v1/health` endpoint
- Server listener on port 3001

### 3.2 `apps/web/src/app/`

**Purpose:** Next.js App Router pages.
**Status:** Default Next.js scaffold (to be replaced on Day 4).
**Contains:**
- `layout.tsx` — root layout
- `page.tsx` — home page (default Next.js page, will become the ScholarForge home screen)
- `globals.css` — Tailwind CSS imports

### 3.3 `packages/shared/src/`

**Purpose:** Shared TypeScript types and zod schemas.
**Status:** Folder structure created, files empty.
**Will contain (Day 4+):**
- `models/` — `NotesDocument.ts`, `Draft.ts`, `FormattedDocument.ts`, etc.
- `schemas/` — zod validation schemas
- `interfaces/` — `IModelClient.ts`, `ITemplateEngine.ts`, etc.

---

## 4. Dependency Boundary Rules (From Day 52)

Still enforced — will be configured in ESLint on Day 4:

| Source | May Import From |
|---|---|
| `apps/web/**` | `packages/shared`, `apps/web/**` (internal) |
| `apps/api/routes/**` | `apps/api/application/**`, `packages/shared` |
| `apps/api/application/**` | `apps/api/ai/**`, `apps/api/formatting/**`, `apps/api/persistence/**`, `apps/api/export/**`, `apps/api/config/**`, `packages/shared` |
| `apps/api/ai/**` | `packages/shared` (NOT `formatting/`, `export/`, `persistence/`) |
| `apps/api/formatting/**` | `packages/shared` (NOT `ai/`, `export/`, `persistence/`) |
| `packages/shared/**` | Nothing (pure types) |

---

## 5. What Gets Created Tomorrow (Day 4)

Based on the updated Implementation Blueprint, Day 4 creates:

| File/Folder | Purpose |
|---|---|
| `packages/shared/src/models/NotesDocument.ts` | Notes data model |
| `packages/shared/src/models/Draft.ts` | Generated draft model |
| `packages/shared/src/schemas/notes.schema.ts` | zod validation |
| `packages/shared/src/interfaces/IModelClient.ts` | Model client contract |
| `apps/api/src/config/ConfigurationService.ts` | Config loader |
| `apps/api/src/config/model.config.json` | Model settings |
| `apps/api/src/ai/client/LlamaCppClient.ts` | llama.cpp adapter |
| `apps/api/src/ai/client/ModelClientFactory.ts` | Backend selector |
| `apps/api/src/ai/prompt/PromptBuilder.ts` | Prompt composition |
| `apps/api/src/ai/cleaning/OutputCleaner.ts` | Output post-processing |
| `apps/api/src/application/workflows/GenerationWorkflow.ts` | Generation orchestration |
| `apps/api/src/routes/generate.routes.ts` | POST /generate/draft endpoint |
| `apps/web/src/app/project/[id]/page.tsx` | Notes Editor UI |
| `tools/debug-model-harness.ts` | Model testing tool |

---

## 6. Verification

**Day 3 checklist (all ✅):**

- [x] Monorepo structure created
- [x] `apps/api` exists with `server.ts`
- [x] `apps/web` exists with Next.js default page
- [x] `packages/shared` exists with folder structure
- [x] `pnpm-workspace.yaml` configured
- [x] `.gitignore` created
- [x] Git initialized + first commit
- [x] Backend runs on `localhost:3001`
- [x] Frontend runs on `localhost:3000`
- [x] `/api/v1/health` endpoint returns JSON

---

**Structure verified.** Ready for Day 4 feature development.
