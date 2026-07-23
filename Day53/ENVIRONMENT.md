# ScholarForge — Environment Configuration

**Project:** ScholarForge V1.0
**Day:** 3 (Foundation Setup)
**OS:** Windows
**Date:** Day 53 of Capstone Program

---

## 1. System Environment

### 1.1 Operating System

| Property | Value |
|---|---|
| OS | Windows |
| Architecture | x64 |
| Project Location | `D:\DevSpace\scholarforge` |

### 1.2 Installed Tools

| Tool | Version | Install Method | Location |
|---|---|---|---|
| Node.js | v24.18.0 | Official installer | System PATH |
| pnpm | 11.16.0 | `npm install -g pnpm` | System PATH |
| Git | 2.52.0.windows.1 | Official installer | System PATH |
| VS Code | Latest | Official installer | System PATH |
| llama.cpp | b10015-12127defd | `winget install ggml.llamacpp` | `C:\Users\asust\AppData\Local\Microsoft\WinGet\Packages\ggml.llamacpp_Microsoft.Winget.Source_8wekyb3d8bbwe\` |

### 1.3 AI Model

| Property | Value |
|---|---|
| Model | Qwen2.5-3B-Instruct |
| Quantization | Q4_K_M |
| Format | GGUF |
| File Location | `D:\DevSpace\Ollama\models\Qwen2.5-3B-Instruct.Q4_K_M.gguf` |
| Size | ~2 GB |
| Backend | llama.cpp (NOT Ollama) |

---

## 2. Project Structure

### 2.1 Monorepo Layout

```
D:\DevSpace\scholarforge\
├── apps/
│   ├── api/                          # Express backend (port 3001)
│   └── web/                          # Next.js frontend (port 3000)
├── packages/
│   └── shared/                       # Shared types + schemas
├── pnpm-workspace.yaml               # Workspace config + build approvals
├── package.json                      # Root package
├── .gitignore
└── pnpm-lock.yaml
```

### 2.2 Workspace Configuration

**File:** `pnpm-workspace.yaml`

```yaml
packages:
  - "apps/*"
  - "packages/*"

onlyBuiltDependencies:
  - esbuild
  - sharp
  - unrs-resolver
```

**Purpose:** 
- `packages` — tells pnpm which folders are workspace packages
- `onlyBuiltDependencies` — pre-approves these packages to run build scripts (pnpm v11 security feature)

---

## 3. Ports

| Service | Port | URL |
|---|---|---|
| Next.js Frontend | 3000 | `http://localhost:3000` |
| Express Backend | 3001 | `http://localhost:3001` |
| llama.cpp (when running) | TBD | `http://localhost:8080` (default, configurable) |

---

## 4. Environment Variables

### 4.1 Current (Day 3)

No `.env` files required yet. All configuration is in code.

### 4.2 Planned (Day 4+)

**File:** `apps/api/.env` (to be created on Day 4)

```env
# Model Configuration
MODEL_BACKEND=llamacpp
MODEL_PATH=D:\DevSpace\Ollama\models\Qwen2.5-3B-Instruct.Q4_K_M.gguf
MODEL_CONTEXT_WINDOW=4096
MODEL_TEMPERATURE=0.7
MODEL_TOP_P=0.9
MODEL_TIMEOUT_MS=30000

# Server Configuration
PORT=3001
CORS_ORIGIN=http://localhost:3000

# llama.cpp Configuration
LLAMA_HOST=localhost
LLAMA_PORT=8080
```

**File:** `apps/web/.env.local` (to be created on Day 4)

```env
# API Configuration
NEXT_PUBLIC_API_URL=http://localhost:3001/api/v1
```

---

## 5. Dependencies

### 5.1 Root (`package.json`)

No dependencies (workspace root only).

### 5.2 `apps/api` (`@scholarforge/api`)

**Dependencies:**
| Package | Version | Purpose |
|---|---|---|
| express | ^4.x | HTTP server |
| cors | ^2.8.5 | Cross-origin resource sharing |
| zod | ^3.x | Runtime schema validation |

**Dev Dependencies:**
| Package | Version | Purpose |
|---|---|---|
| typescript | ^7.0.2 | TypeScript compiler |
| tsx | ^4.23.1 | Run TypeScript directly in dev |
| @types/node | ^26.1.1 | Node.js type definitions |
| @types/express | ^5.0.6 | Express type definitions |
| @types/cors | ^2.8.19 | CORS type definitions |

### 5.3 `apps/web` (`@scholarforge/web`)

**Dependencies:**
| Package | Version | Purpose |
|---|---|---|
| next | 16.2.11 | React framework |
| react | 19.2.4 | UI library |
| react-dom | 19.2.4 | React DOM renderer |
| zustand | 5.0.14 | Client state management |
| @tanstack/react-query | 5.101.4 | Server state management |
| zod | ^3.x | Runtime schema validation |

**Dev Dependencies:**
| Package | Version | Purpose |
|---|---|---|
| typescript | 5.9.3 | TypeScript compiler |
| tailwindcss | 4.3.3 | Utility-first CSS |
| @tailwindcss/postcss | 4.3.3 | Tailwind PostCSS plugin |
| eslint | 9.39.5 | Code linting |
| eslint-config-next | 16.2.11 | Next.js ESLint config |

### 5.4 `packages/shared` (`@scholarforge/shared`)

No dependencies (pure TypeScript types).

---

## 6. Configuration Files

### 6.1 TypeScript Config (`apps/api/tsconfig.json`)

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src"
  }
}
```

### 6.2 Next.js Config (`apps/web/next.config.ts`)

Default Next.js config (no customization yet).

### 6.3 ESLint Config

- `apps/web/eslint.config.mjs` — Next.js default ESLint config
- `apps/api` — uses TypeScript strict mode (no separate ESLint config yet)

### 6.4 Tailwind Config

Tailwind CSS v4 (configured via `postcss.config.mjs`, no separate `tailwind.config.js` needed in v4).

---

## 7. Git Configuration

### 7.1 .gitignore

```
# Dependencies
node_modules/

# Build outputs
apps/web/.next/
apps/api/dist/
*.tsbuildinfo

# Environment variables
.env
.env.local
.env.*.local

# OS files
.DS_Store
Thumbs.db
desktop.ini

# IDE
.vscode/
.idea/
*.swp
*.swo

# ScholarForge session files
*.scholarforge
*.scholarforge.tmp
*.scholarforge.tmp.bak

# Exported documents
*.docx

# Logs
*.log
npm-debug.log*
pnpm-debug.log*
```

### 7.2 Initial Commit

```
Day 3: Project foundation - monorepo skeleton, Express backend, Next.js frontend, Hello World running
```

---

## 8. Development Commands

### 8.1 From Root (`D:\DevSpace\scholarforge`)

| Command | Purpose |
|---|---|
| `pnpm install` | Install all workspace dependencies |
| `pnpm rebuild` | Rebuild native packages (if build scripts were blocked) |

### 8.2 Backend (`apps/api`)

| Command | Purpose |
|---|---|
| `pnpm dev` | Start dev server with hot reload (`tsx watch`) |
| `pnpm build` | Compile TypeScript to `dist/` |
| `pnpm start` | Run compiled server from `dist/` |

### 8.3 Frontend (`apps/web`)

| Command | Purpose |
|---|---|
| `pnpm dev` | Start Next.js dev server |
| `pnpm build` | Production build |
| `pnpm start` | Run production server |
| `pnpm lint` | Run ESLint |

---

## 9. Known Issues & Resolutions

### Issue 1: pnpm `ERR_PNPM_IGNORED_BUILDS`

**Cause:** pnpm v11 blocks build scripts for security.
**Resolution:** Add `onlyBuiltDependencies` to `pnpm-workspace.yaml`, then run `pnpm rebuild`.

### Issue 2: Node.js v24 (not LTS)

**Cause:** User has Node v24.18.0 (current, not LTS).
**Impact:** None for V1 — works fine. If issues arise, downgrade to Node 20 LTS.

### Issue 3: llama.cpp vs Ollama

**Cause:** User chose llama.cpp instead of Ollama.
**Resolution:** Architecture supports both via `IModelClient` interface (ADR-003). `LlamaCppClient` is the primary adapter. Config will use `backend: "llamacpp"`.

---

## 10. Backup & Recovery

### 10.1 What to Back Up

| Item | Location | Priority |
|---|---|---|
| Project source code | `D:\DevSpace\scholarforge\` | High (Git handles this) |
| Qwen2.5-3B model | `D:\DevSpace\Ollama\models\Qwen2.5-3B-Instruct.Q4_K_M.gguf` | Medium (re-downloadable) |
| Session files (future) | `*.scholarforge` | High (user data) |

### 10.2 Recovery

- **Code:** `git clone` from GitHub
- **Model:** Re-download from HuggingFace
- **Dependencies:** `pnpm install`
- **Session files:** Restore from `.scholarforge.tmp.bak` (automatic recovery built into SessionStore)

---

**Environment documented.** Ready for Day 4 feature development.
