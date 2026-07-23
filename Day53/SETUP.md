# ScholarForge — Setup Guide

**Project:** ScholarForge V1.0 — Offline-first AI Academic Writing Assistant
**Day:** 3 (Foundation Setup)
**OS:** Windows
**Date:** Day 53 of Capstone Program

---

## 1. Prerequisites

Before setting up ScholarForge, ensure the following are installed on your Windows machine.

### 1.1 Required Software

| Software | Version Used | Purpose | Install Link |
|---|---|---|---|
| **Node.js** | v24.18.0 | JavaScript runtime (runs Next.js + Express) | https://nodejs.org |
| **pnpm** | 11.16.0 | Package manager (faster than npm, workspace support) | `npm install -g pnpm` |
| **Git** | 2.52.0 | Version control | https://git-scm.com |
| **VS Code** | Latest | Code editor | https://code.visualstudio.com |
| **llama.cpp** | b10015-12127defd | Local AI model runtime (alternative to Ollama) | `winget install ggml.llamacpp` |

### 1.2 Required Model

| Model | File | Location |
|---|---|---|
| **Qwen2.5-3B-Instruct (Q4_K_M)** | `Qwen2.5-3B-Instruct.Q4_K_M.gguf` | `D:\DevSpace\Ollama\models\` |

Download from HuggingFace: https://huggingface.co/Qwen/Qwen2.5-3B-Instruct-GGUF

---

## 2. Installation Steps

### Step 1: Verify Prerequisites

Open a terminal (`Win + R` → type `cmd` → Enter) and run:

```cmd
node --version
pnpm --version
git --version
llama --version
```

All four should return version numbers.

### Step 2: Navigate to Project Location

```cmd
cd D:\DevSpace
```

### Step 3: Create the Project

```cmd
mkdir scholarforge
cd scholarforge
```

### Step 4: Initialize pnpm Workspace

```cmd
pnpm init
```

Create `pnpm-workspace.yaml`:

```yaml
packages:
  - "apps/*"
  - "packages/*"

onlyBuiltDependencies:
  - esbuild
  - sharp
  - unrs-resolver
```

### Step 5: Create Shared Package

```cmd
mkdir packages\shared\src\models
mkdir packages\shared\src\schemas
mkdir packages\shared\src\interfaces
cd packages\shared
pnpm init
```

Rename package to `@scholarforge/shared` in `package.json`.

### Step 6: Create API Backend

```cmd
cd ..\..
mkdir apps\api\src\routes
mkdir apps\api\src\middleware
cd apps\api
pnpm init
pnpm add express cors zod
pnpm add -D typescript tsx @types/node @types/express @types/cors
```

### Step 7: Create Next.js Frontend

```cmd
cd ..\..
pnpm create next-app@latest apps/web --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --use-pnpm
```

### Step 8: Install Frontend Dependencies

```cmd
cd apps\web
pnpm add zustand @tanstack/react-query
pnpm add zod
```

### Step 9: Approve Build Scripts

If pnpm shows `[ERR_PNPM_IGNORED_BUILDS]`, run:

```cmd
pnpm approve-builds
```

Press `a` to select all, then `Enter` to confirm.

---

## 3. Running the Project

### 3.1 Start the Backend (Terminal 1)

```cmd
cd D:\DevSpace\scholarforge\apps\api
pnpm dev
```

**Expected:** `ScholarForge API running on http://localhost:3001`

**Test:** Open `http://localhost:3001/api/v1/health` in your browser — should return JSON.

### 3.2 Start the Frontend (Terminal 2)

```cmd
cd D:\DevSpace\scholarforge\apps\web
pnpm dev
```

**Expected:** Next.js starts on `http://localhost:3000`

**Test:** Open `http://localhost:3000` in your browser — should show the Next.js default page.

---

## 4. VS Code Extensions (Recommended)

Install these in VS Code (Extensions panel → search):

| Extension | Purpose |
|---|---|
| ESLint | JavaScript/TypeScript linting |
| Prettier | Code formatting |
| Tailwind CSS IntelliSense | Tailwind class autocomplete |
| TypeScript Importer | Auto-import TypeScript types |
| GitLens | Git integration enhanced |

---

## 5. Troubleshooting

### Problem: `pnpm approve-builds` says "no packages awaiting approval"

**Fix:** Add `onlyBuiltDependencies` to `pnpm-workspace.yaml` (not `package.json`), then run `pnpm rebuild`.

### Problem: `'llama' is not recognized`

**Fix:** Reinstall via `winget install ggml.llamacpp`, or use the full path: `C:\Users\asust\AppData\Local\Microsoft\WinGet\Packages\ggml.llamacpp_Microsoft.Winget.Source_8wekyb3d8bbwe\llama.exe`

### Problem: Port 3000 or 3001 already in use

**Fix:** Kill the process:
```cmd
netstat -ano | findstr :3000
taskkill /PID <PID> /F
```

---

## 6. Verification Checklist

- [ ] Node.js installed (`node --version` works)
- [ ] pnpm installed (`pnpm --version` works)
- [ ] Git installed (`git --version` works)
- [ ] llama.cpp installed (`llama --version` works)
- [ ] Qwen2.5-3B model downloaded
- [ ] VS Code installed
- [ ] Project folder created at `D:\DevSpace\scholarforge`
- [ ] pnpm workspace configured
- [ ] `packages/shared` created
- [ ] `apps/api` created with Express
- [ ] `apps/web` created with Next.js
- [ ] All dependencies installed
- [ ] Build scripts approved
- [ ] Backend runs on `localhost:3001`
- [ ] Frontend runs on `localhost:3000`
- [ ] `/api/v1/health` returns JSON
- [ ] Git initialized + first commit made

---

**Setup complete.** ScholarForge is ready for feature development on Day 4.
