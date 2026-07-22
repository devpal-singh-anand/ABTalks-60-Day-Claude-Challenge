# ScholarForge — Implementation Blueprint (V1.0, Updated)

**Source of truth:** ScholarForge V1.0 Product Vision + PRD + Software Architecture Document + Day 52 Architecture/Schema/API/Project-Structure/UI-Wireframes deliverables
**Author role:** Senior TPM / Engineering Manager / Agile Sprint Planner
**Status:** Updated blueprint reflecting the Next.js + Express + TypeScript stack locked in on Day 52. Same day-by-day sequence, same scope, same objectives — now implementation-ready with specific libraries, file paths, REST endpoints, and TypeScript interfaces.
**Scope discipline:** No feature added, removed, or reinterpreted. This update specifies the *how* (technologies, endpoints, file paths) so implementation can begin immediately. The original Implementation Blueprint (Days 2–10) remains the authoritative day-by-day plan; this document is a re-skinning, not a redesign.

**What changed from the original blueprint:** The original was deliberately language-agnostic ("pick once here and do not revisit"). Day 52 is the day we pick. Every "language of choice" placeholder is now filled with the specific technology from ARCHITECTURE.md §2. Every "TBD interface" is now the specific REST endpoint from API.md. Every "TBD folder" is now the specific path from PROJECT-STRUCTURE.md.

---

## Day 2

### 🎯 Objective
Stand up the **pnpm workspace monorepo skeleton** exactly as defined in PROJECT-STRUCTURE.md, and build the **Notes Module** (structured research notes editor + validation) — the first functional vertical slice, with zero AI or formatting dependencies yet.

### 📖 Learning Outcomes
- Structuring a pnpm monorepo with `apps/web`, `apps/api`, `packages/shared` and getting workspace path mapping working.
- Defining shared TypeScript types + zod schemas in `packages/shared` that both frontend and backend consume.
- Building a Next.js App Router page with shadcn/ui components for structured input.

### 🛠 Features to Build
- Full monorepo skeleton (all folders from PROJECT-STRUCTURE.md §1, with `.gitkeep` placeholders for empty ones).
- `packages/shared` package with `NotesDocument`, `NoteSection`, `SourceRef`, `SectionType` types + matching zod schemas.
- Next.js Notes Editor page (`/project/[id]`) with: add/edit/remove sections, each with heading, section type dropdown, bullets, source references.
- `NotesValidator` in `apps/api/src/application/validation/NotesValidator.ts` — checks every section has a heading and at least one bullet.
- `POST /api/v1/notes/validate` endpoint wired to the validator.

### 📝 Step-by-Step Implementation Plan
1. `pnpm init` the root workspace. Create `pnpm-workspace.yaml` pointing to `apps/*` and `packages/*`.
2. Create `packages/shared` — `package.json` with name `@scholarforge/shared`, `src/index.ts` barrel export, types in `src/models/`, zod schemas in `src/schemas/`.
3. Define in `packages/shared/src/models/NotesDocument.ts`: `NotesDocument { sections: NoteSection[] }`, `NoteSection { id, heading, bullets: string[], sourceRefs: SourceRef[], sectionType: SectionType }`, `SourceRef { id, author, year, title, source }`, `SectionType` union.
4. Define matching zod schemas in `packages/shared/src/schemas/notes.schema.ts` — `NotesDocumentSchema` that validates the same shape.
5. Create `apps/web` via `pnpm create next-app` (TypeScript, Tailwind, App Router, no src dir → use `src/`). Install shadcn/ui via their CLI, add `button`, `card`, `input`, `textarea`, `select`, `dialog` components.
6. Create `apps/api` — `pnpm init`, install `express`, `cors`, `zod`, `tsx` (for dev), `typescript`. Set up `src/server.ts` with Express + CORS + a `/health` endpoint.
7. In `apps/api/src/application/validation/NotesValidator.ts`, implement `validate(doc: NotesDocument): ValidationResult` returning specific issues (e.g., "Section 2 has no bullets"), not a boolean.
8. In `apps/api/src/routes/notes.routes.ts`, implement `POST /api/v1/notes/validate` — parse body with `NotesDocumentSchema`, run `NotesValidator`, return `{ valid, issues }`.
9. In `apps/web/src/app/project/[id]/page.tsx`, build the Notes Editor UI (see UI-WIREFRAMES.md §3.2). Use shadcn/ui `Card` for each section, `Input` for headings, `Textarea` for bullets, `Select` for section type.
10. Wire the UI to a Zustand store (`apps/web/src/stores/session-store.ts`) holding the current `NotesDocument`. No persistence yet — that's Day 7.
11. Add a "Validate" action that calls `POST /api/v1/notes/validate` and displays issues inline (red border on offending sections).
12. Create the canonical fixture: `apps/api/tests/fixtures/sample_notes/valid_sample.json` — 3–4 sections, at least 2 source references. This fixture is reused by every subsequent day's tests.

### 📂 Folder Changes
Create: `apps/`, `apps/web/` (full Next.js scaffold), `apps/api/` (Express scaffold), `packages/`, `packages/shared/`, `tools/`, `docs/`, `docs/adr/`. Empty folders get `.gitkeep`.

### 📄 Files
| File | Purpose |
|---|---|
| `packages/shared/src/models/NotesDocument.ts` | Core data model for structured notes |
| `packages/shared/src/schemas/notes.schema.ts` | zod schema for runtime validation |
| `apps/api/src/application/validation/NotesValidator.ts` | Structural validation logic |
| `apps/api/src/routes/notes.routes.ts` | POST /api/v1/notes/validate endpoint |
| `apps/api/src/server.ts` | Express app entry |
| `apps/web/src/app/project/[id]/page.tsx` | Notes Editor UI |
| `apps/web/src/stores/session-store.ts` | Zustand store for current session state |
| `apps/api/tests/fixtures/sample_notes/valid_sample.json` | Canonical test fixture |
| `apps/api/tests/unit/validation/NotesValidator.test.ts` | Vitest unit tests for validator |
| `docs/adr/ADR-000-day2-kickoff.md` | Records pnpm/Next.js/Express/Vitest decisions |

### 📦 Libraries / Packages
- `pnpm` (workspace manager)
- `next`, `react`, `react-dom` (frontend)
- `tailwindcss`, `shadcn/ui` components (styling)
- `zustand` (client state)
- `express`, `cors` (backend)
- `zod` (validation, shared between frontend and backend)
- `tsx` (run TypeScript directly in dev)
- `vitest` (test runner)
- No AI, DOCX, or persistence libraries yet.

### 🔗 Integrations
- `packages/shared` is consumed by both `apps/web` and `apps/api` via pnpm workspace path mapping. Both apps' `tsconfig.json` extend `../../tsconfig.base.json`.
- `apps/web` calls `apps/api` via `fetch('http://localhost:3001/api/v1/notes/validate', ...)`. The base URL is in `apps/web/src/lib/constants.ts`.
- `NotesDocument` in `packages/shared` becomes the contract every later layer consumes — treat changes to this schema as expensive starting tomorrow.

### 🧪 Testing Tasks
- **Unit (Vitest):** `NotesValidator` against: empty document, section with no heading, section with no bullets, valid document, document with duplicate section IDs.
- **Manual:** Add 3 sections through the UI, remove one, edit a heading, verify the Zustand store reflects correctly.
- **Edge cases:** Very long bullet text, special characters in headings, a section with 0 source references (must be allowed — sources are optional per PRD).
- **Expected outputs:** `POST /api/v1/notes/validate` returns `{ valid: true, issues: [] }` for the fixture; returns specific issues for each broken case.

### 🐞 Common Bugs
- **Bug:** Frontend and backend disagree on the `NotesDocument` shape. **Root cause:** types duplicated instead of imported from `@scholarforge/shared`. **Recovery:** never redefine a shared type — always import from `@scholarforge/shared`.
- **Bug:** CORS error when frontend calls backend. **Root cause:** Express CORS config doesn't include `http://localhost:3000`. **Recovery:** add `http://localhost:3000` to the CORS origin list (see API.md §9).
- **Bug:** zod schema and TypeScript interface drift. **Root cause:** manually maintained separately. **Recovery:** use `z.infer<typeof NotesDocumentSchema>` to derive the TypeScript type from the zod schema — single source of truth.

### ✅ End-of-Day Checklist
- [ ] Monorepo skeleton exists and matches PROJECT-STRUCTURE.md §1.
- [ ] `packages/shared` exports `NotesDocument` types + zod schemas, consumed by both apps.
- [ ] `NotesValidator` implemented with specific error messages, covered by Vitest tests.
- [ ] `POST /api/v1/notes/validate` endpoint working.
- [ ] Notes Editor UI functional at `localhost:3000/project/[id]`.
- [ ] Sample fixture (`valid_sample.json`) created and passes validation.
- [ ] All Day 2 Vitest tests green.

### ➡️ Handoff Notes
The monorepo skeleton exists and matches the approved structure. The Notes Module is complete, tested, and produces a validated `NotesDocument` — this is the only artifact Day 3 needs as an input assumption. Day 3 builds the Configuration Layer and AI Layer scaffolding completely independently, consuming `tests/fixtures/sample_notes/valid_sample.json` as its test input.

---

## Day 3

### 🎯 Objective
Build the **Configuration Layer** in full, and the **AI Layer's connectivity skeleton**: `IModelClient` interface plus both `OllamaClient` and `LlamaCppClient` implementations, validated against the real local Qwen2.5-3B model with a trivial hardcoded prompt. By end of day, the riskiest unknown (does Qwen2.5-3B run acceptably on this 4GB GPU) must be empirically answered.

### 📖 Learning Outcomes
- Designing a backend-agnostic adapter interface (`IModelClient`) in TypeScript with zod-validated config.
- Practical quantization/context-window tuning for a 3B model under 4GB VRAM.
- Calling the Ollama HTTP API (`localhost:11434`) from Express via native `fetch` with `AbortController` for timeout enforcement.

### 🛠 Features to Build
- `ConfigurationService` in `apps/api/src/config/ConfigurationService.ts` — loads + validates all config at startup, exposes typed getters.
- `model.config.json` schema (backend, model, quantization, context, timeout) validated by zod on load.
- `IModelClient` interface in `packages/shared/src/interfaces/IModelClient.ts`.
- `OllamaClient` and `LlamaCppClient` in `apps/api/src/ai/client/`.
- `ModelClientFactory` — selects implementation based on config.
- `GET /api/v1/health` endpoint extended to include model reachability check.
- `GET /api/v1/config/model` endpoint.
- A debug harness in `tools/debug-model-harness.ts` that loads config, creates a client, sends a fixed test prompt, logs response + latency.

### 📝 Step-by-Step Implementation Plan
1. Install and verify Ollama locally: `ollama pull qwen2.5:3b-instruct-q4_K_M`, then `ollama run qwen2.5:3b-instruct-q4_K_M "test prompt"` — confirm it works outside the app first.
2. In `apps/api/src/config/model.config.json`, define: `{ backend: "ollama", modelName: "qwen2.5:3b-instruct-q4_K_M", quantization: "Q4_K_M", contextWindow: 4096, temperature: 0.7, topP: 0.9, timeoutMs: 30000, ollamaBaseUrl: "http://localhost:11434" }`.
3. Implement `ConfigurationService.load()` — reads `model.config.json`, validates with zod, throws a specific error if anything is missing/malformed.
4. In `packages/shared/src/interfaces/IModelClient.ts`, define `IModelClient { generate(prompt: string, options: GenerateOptions): Promise<string> }` where `GenerateOptions` includes `timeoutMs`, `temperature`, `topP`.
5. In `apps/api/src/ai/client/OllamaClient.ts`, implement `IModelClient` using native `fetch` to POST to `${ollamaBaseUrl}/api/generate`. Use `AbortController` with `setTimeout` for timeout enforcement — actually abort the fetch, don't just race a timer.
6. In `apps/api/src/ai/client/LlamaCppClient.ts`, implement `IModelClient` by spawning a local llama.cpp process (or calling a local llama.cpp server if one is running). This can be thinner than OllamaClient — it needs to exist and compile, but the demo backend is Ollama.
7. In `apps/api/src/ai/client/ModelClientFactory.ts`, implement `create(config: ModelConfig): IModelClient` — switches on `config.backend`, returns the right client.
8. Extend `GET /api/v1/health` to do a pre-flight check: ping `${ollamaBaseUrl}/api/tags` (Ollama's list-models endpoint) to confirm the model is reachable. Return `modelBackend.reachable: true/false` with the error message if false.
9. Add `GET /api/v1/config/model` returning the current config (read-only).
10. In `tools/debug-model-harness.ts`, write a standalone script: load config → create client → send "Write a one-paragraph introduction about AI in education." → log response, latency, and (if available) `nvidia-smi` output for VRAM usage.
11. Run the harness 3 times with different `contextWindow` values (2048, 4096, 8192) and different quantizations if available. Record results in `docs/adr/ADR-003-model-backend-tuning.md`.
12. Lock in the final `model.config.json` defaults based on what stayed under 4GB VRAM with acceptable latency.

### 📂 Folder Changes
Populate: `apps/api/src/config/`, `apps/api/src/ai/client/`, `packages/shared/src/interfaces/`, `packages/shared/src/models/Config.ts`, `tools/`.

### 📄 Files
| File | Purpose |
|---|---|
| `apps/api/src/config/ConfigurationService.ts` | Loads/validates all config, exposes typed getters |
| `apps/api/src/config/model.config.json` | Backend, model, quantization, context, timeout settings |
| `packages/shared/src/interfaces/IModelClient.ts` | Contract every backend adapter must satisfy |
| `packages/shared/src/models/Config.ts` | `ModelConfig` type |
| `packages/shared/src/schemas/config.schema.ts` | zod schema for `ModelConfig` |
| `apps/api/src/ai/client/OllamaClient.ts` | Ollama-backed implementation |
| `apps/api/src/ai/client/LlamaCppClient.ts` | llama.cpp-backed implementation |
| `apps/api/src/ai/client/ModelClientFactory.ts` | Config-driven selector |
| `tools/debug-model-harness.ts` | Manual smoke-test tool for latency/memory validation |
| `docs/adr/ADR-003-model-backend-tuning.md` | Records empirical quantization/context findings |

### 📦 Libraries / Packages
- Native `fetch` (Node 20 has it built in) — no `node-fetch` or `axios` needed.
- `zod` (already installed Day 2) — for config validation.
- Ollama must be installed and running as a local service before `OllamaClient` can be tested.

### 🔗 Integrations
- `ConfigurationService` becomes a dependency of every layer from here forward — no module reads a config file directly except through it.
- `IModelClient` is the seam the entire rest of the AI Layer (Day 4's `PromptBuilder`/`GenerationWorkflow`) will depend on — today's job is to make this seam boring and reliable.

### 🧪 Testing Tasks
- **Unit (Vitest):** `ConfigurationService` against a valid config, a missing-field config, a malformed-JSON config.
- **Unit:** `ModelClientFactory` returns `OllamaClient` when `backend: "ollama"`, `LlamaCppClient` when `backend: "llamacpp"`.
- **Integration:** Real call through `ModelClientFactory` → `OllamaClient` → real Ollama → Qwen2.5-3B, with a fixed test prompt, confirming a non-empty response within 30s timeout.
- **Manual:** Stop the Ollama daemon, call `GET /api/v1/health`, confirm it returns 503 with a specific error.
- **Edge cases:** Timeout enforced — send a prompt that takes too long, confirm `AbortController` actually cancels the fetch (check no zombie connections).

### 🐞 Common Bugs
- **Bug:** Ollama silently OOMs on too-large a context window. **Root cause:** `contextWindow` set higher than 4GB allows for Q4_K_M. **Recovery:** monitor VRAM during the harness run; reduce `contextWindow` to 4096 or 2048 until stable.
- **Bug:** `OllamaClient` works from `curl` but not from the app. **Root cause:** base-URL mismatch (e.g., `http://localhost:11434` vs `http://127.0.0.1:11434`) or daemon not started. **Recovery:** add the pre-flight check in `/health` (step 8).
- **Bug:** Timeout doesn't actually cancel the underlying fetch. **Root cause:** timeout implemented as `Promise.race` without aborting the fetch. **Recovery:** use `AbortController` — `const controller = new AbortController(); setTimeout(() => controller.abort(), timeoutMs); fetch(url, { signal: controller.signal })`.

### ✅ End-of-Day Checklist
- [ ] `ConfigurationService` implemented, validated, tested.
- [ ] `IModelClient` interface defined and stable in `packages/shared`.
- [ ] Both `OllamaClient` and `LlamaCppClient` implemented (Ollama fully working; llama.cpp compiles).
- [ ] Empirical quantization/context testing completed and documented in ADR-003.
- [ ] Debug harness generates a real response from Qwen2.5-3B within the 4GB budget.
- [ ] `GET /api/v1/health` reports model reachability.
- [ ] `GET /api/v1/config/model` returns the locked-in config.

### ➡️ Handoff Notes
The model connectivity seam is proven end-to-end. Day 4 builds directly on top of `IModelClient` — it adds `PromptBuilder`, `VoiceProfile`, `OutputCleaner`, and `GenerationWorkflow` on top of this now-stable foundation, with real section-based prompts.

---

## Day 4

### 🎯 Objective
Build the full **prose-generation pipeline**: `PromptBuilder`, `VoiceProfile`, `OutputCleaner`, and `GenerationWorkflow`, producing a real end-to-end `Draft` from the Day 2 fixture, using the Day 3 `IModelClient`. First day the "notes → generated academic prose" core value proposition becomes real.

### 📖 Learning Outcomes
- Prompt engineering for small (3B-class) models: keeping per-call context small and instruction-dense.
- Designing a voice/style injection mechanism that's honest about being approximate.
- Building resilient generation loops with retry-on-empty and partial-failure tolerance.

### 🛠 Features to Build
- `VoiceProfile` type in `packages/shared` + `voice_profile.default.json` config file.
- `PromptBuilder` in `apps/api/src/ai/prompt/` — composes one prompt per section, injecting notes + voice + section template.
- Section prompt template JSON files in `apps/api/src/ai/prompt/SectionPromptTemplates/`.
- `OutputCleaner` in `apps/api/src/ai/cleaning/` — strips artifacts, validates non-empty/min-length, triggers retry.
- `GenerationWorkflow` in `apps/api/src/application/workflows/` — orchestrates the notes → per-section generate → clean → assembled `Draft` loop.
- `Draft`/`DraftSection` types in `packages/shared`.
- `POST /api/v1/generate/draft` endpoint.
- `POST /api/v1/generate/draft/stream` (SSE variant) for real-time progress.
- Minimal Draft View in the frontend (`/project/[id]/draft`).

### 📝 Step-by-Step Implementation Plan
1. In `packages/shared/src/models/Draft.ts`, define `Draft { sections: DraftSection[], failedSectionIds: string[], generatedAt: string, modelConfigSnapshot: ModelConfigSnapshot }` and `DraftSection { id, sourceSectionId, heading, content, generationMetadata: { promptTokensEstimate, responseTimeMs, retries } }`.
2. In `packages/shared/src/models/SessionState.ts`, define `VoiceProfile { profileId, profileName, formality, sentenceLengthTendency, personPreference, hedgingLevel, exemplarSnippets: string[] }`.
3. In `apps/api/src/config/voice_profile.default.json`, create the default academic profile: `{ profileId: "default", profileName: "Default Academic", formality: "formal", sentenceLengthTendency: "varied", personPreference: "third_person", hedgingLevel: "moderate", exemplarSnippets: [...] }`.
4. In `apps/api/src/ai/prompt/SectionPromptTemplates/`, create JSON files per section type: `introduction.json`, `literature_review.json`, `methodology.json`, `results.json`, `discussion.json`, `conclusion.json`. Each has `{ systemInstruction, taskInstruction, voiceInjectionPoint, contextBudgetChars, minOutputLengthChars, retryInstructionSuffix }`.
5. In `apps/api/src/ai/prompt/PromptBuilder.ts`, implement `build(section: NoteSection, voiceProfile: VoiceProfile, sectionTemplate: SectionPromptTemplate): string` — composes the prompt, enforces `contextBudgetChars` (truncate notes if they exceed budget).
6. In `apps/api/src/ai/cleaning/OutputCleaner.ts`, implement `clean(rawText: string): { cleanedText: string, isAcceptable: boolean }` — strip markdown artifacts (`**`, `##`), strip boilerplate/refusals, repair truncation (drop incomplete trailing sentence), normalize whitespace, check `minOutputLengthChars`.
7. In `apps/api/src/application/workflows/GenerationWorkflow.ts`, implement `generateDraft(notes: NotesDocument, voice: VoiceProfile, modelClient: IModelClient, config: ModelConfig): Promise<Draft>`:
   - Iterate `notes.sections` in order.
   - For each: `PromptBuilder.build()` → `modelClient.generate()` → `OutputCleaner.clean()` → if unacceptable, retry once with `retryInstructionSuffix` → assemble `DraftSection`.
   - On failure (timeout, OOM, persistent unacceptable output): log, add section ID to `failedSectionIds`, continue.
8. In `apps/api/src/routes/generate.routes.ts`, implement `POST /api/v1/generate/draft` — parse body with zod, validate notes, call `GenerationWorkflow`, return `Draft`.
9. Implement `POST /api/v1/generate/draft/stream` using SSE — emit `section_started`, `section_completed`, `section_failed`, `done` events.
10. In `apps/web/src/app/project/[id]/draft/page.tsx`, build the Draft View (see UI-WIREFRAMES.md §3.3). Add a TanStack Query mutation hook (`apps/web/src/hooks/useGenerateDraft.ts`) that calls the SSE endpoint and updates the UI per-section.
11. Wire a "Generate Draft" button in the Notes Editor that navigates to the Draft View and triggers generation.
12. Run the full pipeline against the Day 2 fixture; manually review output quality; iterate prompt templates.

### 📂 Folder Changes
Populate: `apps/api/src/ai/prompt/`, `apps/api/src/ai/prompt/SectionPromptTemplates/`, `apps/api/src/ai/cleaning/`, `apps/api/src/application/workflows/`, `apps/api/src/config/voice_profile.default.json`, `apps/web/src/app/project/[id]/draft/`, `apps/web/src/hooks/`.

### 📄 Files
| File | Purpose |
|---|---|
| `packages/shared/src/models/Draft.ts` | Generated-content data model |
| `packages/shared/src/models/SessionState.ts` | VoiceProfile + SessionState types |
| `apps/api/src/config/voice_profile.default.json` | Default style config |
| `apps/api/src/ai/prompt/SectionPromptTemplates/*.json` | Data-driven instructional prompts |
| `apps/api/src/ai/prompt/PromptBuilder.ts` | Prompt composition logic |
| `apps/api/src/ai/cleaning/OutputCleaner.ts` | Post-processing/validation |
| `apps/api/src/application/workflows/GenerationWorkflow.ts` | Orchestrates the full notes→draft use case |
| `apps/api/src/routes/generate.routes.ts` | POST /generate/draft + /generate/draft/stream + /generate/review |
| `apps/web/src/app/project/[id]/draft/page.tsx` | Draft View UI |
| `apps/web/src/hooks/useGenerateDraft.ts` | TanStack Query + SSE hook |
| `apps/api/tests/unit/ai/OutputCleaner.test.ts` | Vitest cleaner tests |
| `apps/api/tests/integration/GenerationWorkflow.test.ts` | Mocked-IModelClient orchestration test |

### 📦 Libraries / Packages
- No new external libraries. Consider `eventsource` for SSE on the frontend if native `EventSource` doesn't fit, but native is preferred.
- No NLP/tokenization library — use a conservative character-length heuristic for context budgeting. Only add a tokenizer if testing shows it's necessary.

### 🔗 Integrations
- `GenerationWorkflow` depends on `IModelClient` (Day 3), `NotesValidator`'s validated `NotesDocument` (Day 2), and `ConfigurationService` for voice/model settings.
- `Draft` produced today has no knowledge of template/formatting concerns — pure content. This is the seam Day 5 consumes unchanged.

### 🧪 Testing Tasks
- **Unit:** `OutputCleaner` against: clean text, text with markdown artifacts, text truncated mid-sentence, empty string, whitespace-only string.
- **Integration:** `GenerationWorkflow` against a **mocked** `IModelClient` (deterministic fixed responses) — verify retry logic, partial-failure handling, without the real model.
- **Manual (real model):** Run the full pipeline against the fixture; read every generated section for coherence and voice fit.
- **Edge cases:** A section with very sparse notes (one bullet) — does the model produce usable output?
- **Expected outputs:** A complete `Draft` with one non-empty `DraftSection` per `NoteSection`, readable prose reflecting the voice profile's formality.

### 🐞 Common Bugs
- **Bug:** Generated prose ignores the voice profile. **Root cause:** voice instructions buried too deep in the prompt. **Recovery:** move voice instructions to the very start of the prompt (per `voiceInjectionPoint: "start"`); strengthen with 1–2 concrete exemplar sentences.
- **Bug:** `OutputCleaner` rejects good output because `minOutputLengthChars` is miscalibrated. **Recovery:** calibrate empirically against real section lengths; store as a config value, not a magic number.
- **Bug:** SSE stream drops connections on slow sections. **Root cause:** no keepalive ping during long generations. **Recovery:** emit a `keepalive` event every 5s during generation.

### ✅ End-of-Day Checklist
- [ ] `VoiceProfile` model + default config implemented.
- [ ] `PromptBuilder` producing well-formed, context-budget-respecting prompts.
- [ ] `OutputCleaner` implemented and unit-tested.
- [ ] `GenerationWorkflow` orchestrates full notes→draft generation with partial-failure tolerance.
- [ ] `POST /api/v1/generate/draft` and `/generate/draft/stream` endpoints working.
- [ ] Real end-to-end run against the fixture produces a complete, readable `Draft`.
- [ ] Draft View UI shows generated content with section-by-section progress.

### ➡️ Handoff Notes
The core generative pipeline is real and working. No formatting or export exists yet — the `Draft` is displayed as plain, unstyled text. Day 5 consumes this exact `Draft` object unchanged and builds the `TemplateEngine` + IEEE `TemplateDefinition` to turn it into a `FormattedDocument`.

---

## Day 5

### 🎯 Objective
Build the generic, data-driven **Template Engine** and ship the **first complete `TemplateDefinition`: IEEE (US Letter)**. By end of day, a `Draft` becomes a fully-resolved `FormattedDocument` with zero LLM involvement in any formatting decision.

### 📖 Learning Outcomes
- Building a rules engine driven entirely by JSON data rather than per-case code branches.
- Designing a citation-formatting strategy pattern that generalizes across in-text styles.
- Enforcing the deterministic/AI seam in practice: Formatting Layer tests never require a model.

### 🛠 Features to Build
- `FormattedDocument`, `StyleSheet`, `FormattedReference` types in `packages/shared`.
- `TemplateDefinition` schema (zod) in `packages/shared`.
- `ieee_us_letter.json` data file in `apps/api/src/formatting/templates/`.
- `ITemplateEngine` interface in `packages/shared`.
- `TemplateEngine`, `LayoutResolver`, `CitationFormatter`, `TemplateRegistry` in `apps/api/src/formatting/engine/`.
- `POST /api/v1/format/apply` endpoint.
- `GET /api/v1/format/templates/:id` endpoint.
- `GET /api/v1/config/templates` endpoint (lists all available templates).

### 📝 Step-by-Step Implementation Plan
1. In `packages/shared/src/models/FormattedDocument.ts`, define `FormattedDocument { sections: FormattedSection[], styleSheet: StyleSheet, references: FormattedReference[], templateSelection: TemplateSelection }`, plus `StyleSheet`, `FormattedSection`, `FormattedReference`, `TemplateSelection`.
2. In `packages/shared/src/schemas/`, define `TemplateDefinitionSchema` (zod) validating the JSON shape from SCHEMA.md §3.3.
3. In `apps/api/src/formatting/templates/ieee_us_letter.json`, author the IEEE US Letter template per the example in SCHEMA.md §3.3.
4. In `apps/api/src/formatting/engine/TemplateRegistry.ts`, implement loading all `*.json` files from `templates/` at startup, validating each with `TemplateDefinitionSchema`, exposing a lookup by `id` (e.g., `"ieee_us_letter"`).
5. In `apps/api/src/formatting/engine/LayoutResolver.ts`, implement `resolve(draft: Draft, template: TemplateDefinition): LayoutInstructions` — pure function translating `StyleSheet` data + section structure into layout instructions. **No conditional logic keyed on standard name** — everything comes from the data file.
6. In `apps/api/src/formatting/engine/CitationFormatter.ts`, implement a strategy registry: `{ numeric: formatNumeric, authorDate: formatAuthorDate }`. `formatNumeric(sourceRef, number)` → `[1] Smith, "Title," Source, Year.` Only `numeric` needs to work today; `authorDate` stubbed for Day 9.
7. In `apps/api/src/formatting/engine/TemplateEngine.ts`, implement `apply(draft: Draft, selection: TemplateSelection): FormattedDocument` — loads template via registry, resolves layout, formats citations, assembles the `FormattedDocument`. Also implement `validate(draft, selection): ValidationResult` — checks required fields (e.g., title page fields) before formatting.
8. In `apps/api/src/routes/format.routes.ts`, implement `POST /api/v1/format/apply` (parse body, call `TemplateEngine.apply`, return `FormattedDocument`), `GET /api/v1/format/templates/:id` (return template definition), `GET /api/v1/config/templates` (list all template metadata).
9. Write the full Vitest suite for `TemplateEngine`/`LayoutResolver`/`CitationFormatter` — **none of these tests require the model to be running**. This is the strongest test coverage in the project.

### 📂 Folder Changes
Populate: `apps/api/src/formatting/engine/`, `apps/api/src/formatting/templates/`. Add `FormattedDocument`/`StyleSheet`/`FormattedReference`/`TemplateSelection` to `packages/shared/src/models/`.

### 📄 Files
| File | Purpose |
|---|---|
| `packages/shared/src/models/FormattedDocument.ts` | Structural output of Formatting Layer |
| `packages/shared/src/schemas/formatted-document.schema.ts` | zod schemas for FormattedDocument + TemplateDefinition |
| `apps/api/src/formatting/templates/ieee_us_letter.json` | First complete template definition |
| `packages/shared/src/interfaces/ITemplateEngine.ts` | Contract for the formatting engine |
| `apps/api/src/formatting/engine/TemplateEngine.ts` | Generic engine implementation |
| `apps/api/src/formatting/engine/LayoutResolver.ts` | Data-driven layout resolution |
| `apps/api/src/formatting/engine/CitationFormatter.ts` | Strategy-based citation formatting |
| `apps/api/src/formatting/engine/TemplateRegistry.ts` | Loads + caches template files |
| `apps/api/src/routes/format.routes.ts` | POST /format/apply, GET /format/templates/:id, GET /config/templates |
| `apps/api/tests/unit/formatting/TemplateEngine.test.ts` | Core deterministic-layer test suite |
| `apps/api/tests/unit/formatting/CitationFormatter.test.ts` | Citation strategy tests |

### 📦 Libraries / Packages
- `zod` (already installed) — for `TemplateDefinition` schema validation.
- No AI or DOCX libraries — this layer is pure, dependency-light logic.

### 🔗 Integrations
- `TemplateEngine` depends on `ConfigurationService`'s template registry and consumes `Draft` objects from Day 4 unchanged.
- Output (`FormattedDocument`) is the exact input Day 6's `DocxExporter` will consume — today's job is to make that object complete and self-sufficient.

### 🧪 Testing Tasks
- **Unit:** `LayoutResolver` against IEEE — verify margins/font/spacing/heading levels resolve exactly as in the JSON file. `CitationFormatter` numeric strategy against 2–3 sample `SourceRef` entries.
- **Integration:** Full `TemplateEngine.apply()` against the Day 4 real generated `Draft` + IEEE selection, verifying a complete `FormattedDocument` with no missing sections.
- **Edge cases:** A `Draft` missing a section IEEE expects (`validate()` catches it with a specific message); a `SourceRef` missing a field the citation format needs (graceful fallback or explicit error).
- **Expected outputs:** A `FormattedDocument` whose `styleSheet` exactly matches `ieee_us_letter.json`'s values.

### 🐞 Common Bugs
- **Bug:** `LayoutResolver` hardcodes an IEEE-specific rule. **Root cause:** convenience shortcut. **Recovery:** grep the resolver for any literal standard name string — there should be none. Move the value into the JSON.
- **Bug:** Citation numbering doesn't stay stable if sections are reordered. **Root cause:** numbering per-section instead of globally. **Recovery:** compute the full reference list and numbering as a single pass over the entire `Draft`.

### ✅ End-of-Day Checklist
- [ ] `FormattedDocument`/`StyleSheet` models implemented in `packages/shared`.
- [ ] `ieee_us_letter.json` complete and schema-valid.
- [ ] `TemplateEngine`, `LayoutResolver`, `CitationFormatter`, `TemplateRegistry` implemented with zero standard-specific conditional branches.
- [ ] Full IEEE formatting pipeline runs against the Day 4 draft, end-to-end, with no model call.
- [ ] Vitest suite for the Formatting Layer passes and requires no running model.
- [ ] `POST /api/v1/format/apply`, `GET /api/v1/format/templates/:id`, `GET /api/v1/config/templates` endpoints working.

### ➡️ Handoff Notes
The deterministic Formatting Layer is real, tested, and proven against IEEE end-to-end. Day 6 takes today's `FormattedDocument` output and renders it into a real `.docx` file — it should not need to ask the Formatting Layer for any additional information.

---

## Day 6

### 🎯 Objective
Build the **Export Layer**: `DocxExporter`/`DocxRenderer`, producing a real, valid, openable `.docx` file from the IEEE `FormattedDocument`. By end of day, the full pipeline **Notes → Generation → IEEE Formatting → DOCX Export** works end-to-end for the first time — the first fully demoable slice.

### 📖 Learning Outcomes
- Mapping an abstract structural document tree to concrete OOXML using the `docx` npm library.
- Handling structured non-prose content (tables, captions, references) without routing them through the LLM.
- Validating exported file integrity programmatically.

### 🛠 Features to Build
- `IExporter` interface in `packages/shared`.
- `DocxExporter` in `apps/api/src/export/` — orchestrates export.
- `DocxRenderer` — `FormattedDocument` → `docx` library document object.
- `StyleMapper` — `StyleSheet` → `docx` style definitions.
- Reference-list rendering.
- Header/footer rendering per IEEE template.
- `POST /api/v1/export/docx` endpoint.
- Export View in the frontend (`/project/[id]/export`).

### 📝 Step-by-Step Implementation Plan
1. Install the `docx` npm library in `apps/api`: `pnpm --filter @scholarforge/api add docx`.
2. In `packages/shared/src/interfaces/IExporter.ts`, define `IExporter { export(document: FormattedDocument, path: string, metadata: ExportMetadata): Promise<ExportResult> }`.
3. In `apps/api/src/export/StyleMapper.ts`, implement `map(styleSheet: StyleSheet): docx.IStylesOptions` — translate margins (inches → twips: 1in = 1440 twips), font, spacing, page size into the `docx` library's style API. Add unit tests for unit conversion.
4. In `apps/api/src/export/DocxRenderer.ts`, implement `render(document: FormattedDocument, metadata: ExportMetadata): docx.Document` — iterate `FormattedDocument.sections`, apply heading levels, render body paragraphs, append the references section from `FormattedDocument.references`.
5. Implement header/footer rendering per IEEE's rules (page number top-right) as a discrete rendering step.
6. Implement basic structured-content passthrough: if a `NoteSection` includes tabular data, confirm it survives as a native DOCX table. Add a table to the sample fixture today to exercise this path.
7. In `apps/api/src/export/DocxExporter.ts`, implement `export()` — call `DocxRenderer`, write to disk via `docx.Packer.toBuffer()` + `fs.writeFile()`, validate the file is parseable (re-read it), return `ExportResult { filePath, sizeBytes, exportedAt }`.
8. In `apps/api/src/routes/export.routes.ts`, implement `POST /api/v1/export/docx` — parse body, validate output path (absolute, ends in `.docx`, parent dir exists), call `DocxExporter.export()`, return result.
9. In `apps/web/src/app/project/[id]/export/page.tsx`, build the Export View (see UI-WIREFRAMES.md §3.5). Add the TanStack Query mutation `useExportDocx` that calls the endpoint and shows a success toast on completion.
10. Wire a full "Generate → Format → Export" action into the UI, chaining Day 4 → Day 5 → today's export into one user-facing flow.
11. Manually open the resulting `.docx` in LibreOffice Writer (and Microsoft Word if available) — visually verify margins, font, heading styles, and reference list match IEEE US Letter expectations.

### 📂 Folder Changes
Populate: `apps/api/src/export/`. Add `apps/web/src/app/project/[id]/export/`, `apps/web/src/hooks/useExportDocx.ts`.

### 📄 Files
| File | Purpose |
|---|---|
| `packages/shared/src/interfaces/IExporter.ts` | Contract for all export formats |
| `apps/api/src/export/DocxExporter.ts` | Orchestrates DOCX export |
| `apps/api/src/export/DocxRenderer.ts` | FormattedDocument → docx.Document mapping |
| `apps/api/src/export/StyleMapper.ts` | StyleSheet → docx style translation |
| `apps/api/src/routes/export.routes.ts` | POST /export/docx endpoint |
| `apps/web/src/app/project/[id]/export/page.tsx` | Export View UI |
| `apps/web/src/hooks/useExportDocx.ts` | TanStack Query mutation |
| `apps/api/tests/integration/end_to_end_ieee_pipeline/full_pipeline.test.ts` | First full Notes→DOCX integration test |

### 📦 Libraries / Packages
- `docx` (npm, by dolanmiu) — pure JS DOCX generation, no native deps, runs fully offline, no paid license.
- Verify the library runs fully offline (some DOCX libraries fetch template assets remotely on first use — reject any that do).

### 🔗 Integrations
- `DocxExporter` depends only on `FormattedDocument` — zero dependency on `IModelClient` or any AI-layer component.
- Verify this independence explicitly: run the export test suite with the Ollama daemon stopped, using a cached `Draft` fixture. The export must still work.

### 🧪 Testing Tasks
- **Unit:** `StyleMapper` against the IEEE `StyleSheet` — verify each field maps to the correct `docx` style property. Unit conversion (inches → twips) tested separately.
- **Integration:** Full pipeline test — sample notes → real generation (or cached `Draft` fixture) → IEEE formatting → DOCX export → verify the file is valid (parseable as well-formed DOCX, not just "exists").
- **Manual:** Open the exported file in LibreOffice Writer (and Microsoft Word if available) — catch renderer-specific compatibility issues.
- **Edge cases:** Very long heading (line-wrap); 10+ references (pagination); empty section (skip or render empty heading — decide explicitly).
- **Expected outputs:** A `.docx` file that opens without repair prompts, with correct 0.75"/1"/0.625" margins, Times New Roman 10pt, correct heading hierarchy, and a numerically-ordered IEEE reference list.

### 🐞 Common Bugs
- **Bug:** Exported DOCX opens with a "needs repair" warning. **Root cause:** malformed OOXML from incomplete style mapping. **Recovery:** ensure every `StyleSheet` field has a complete `docx` style entry.
- **Bug:** Margins correct in LibreOffice but wrong in Word. **Root cause:** unit conversion error (inches vs twips). **Recovery:** add an explicit `inchesToTwips()` utility with unit tests.
- **Bug:** References list numbering doesn't match in-text citations. **Root cause:** Day 5's numbering and today's rendering order diverged. **Recovery:** confirm `DocxRenderer` renders `FormattedDocument.references` in the exact order from Day 5 — never re-sort.

### ✅ End-of-Day Checklist
- [ ] `IExporter`/`DocxExporter`/`DocxRenderer`/`StyleMapper` implemented.
- [ ] `POST /api/v1/export/docx` endpoint working.
- [ ] Full Notes→Generation→IEEE-Formatting→DOCX pipeline runs end-to-end.
- [ ] Exported file opens cleanly (no repair prompts) in at least one word processor.
- [ ] Margins, fonts, headings, and references visually verified against IEEE US Letter.
- [ ] Export layer's independence from AI layer verified (export test passes with model stopped).

### ➡️ Handoff Notes
**This is the first day the product's core acceptance criterion is met**: "given structured notes and a selected template standard, the system produces a complete draft with that standard's formatting correctly applied" — for IEEE, fully. This is a legitimate demo-ready milestone. No session persistence exists yet. Day 7 adds session save/load/autosave on top of this now-working pipeline.

---

## Day 7

### 🎯 Objective
Build the full **Session/Persistence Layer**: manual save, autosave, and reload-with-recovery, so that a user's notes, draft, template selection, and voice profile survive an app restart without data loss (FR-6/FR-7).

### 📖 Learning Outcomes
- Designing atomic, crash-safe local file writes (temp-file-then-rename).
- Schema-versioning a serialization format from day one.
- Recovery-oriented error handling for corrupt save files.

### 🛠 Features to Build
- `ISessionStore` interface in `packages/shared`.
- `SessionStore` in `apps/api/src/persistence/` — save (atomic write) + load (validate + recover).
- `AutosaveScheduler` logic (frontend-triggered via debounced `POST /session/save`).
- `session_schema_v1` zod schema.
- `POST /api/v1/session/save` endpoint.
- `POST /api/v1/session/load` endpoint.
- `GET /api/v1/session/recent` endpoint.
- Save/Load UI in the frontend (Save dialog, Open dialog, autosave indicator).

### 📝 Step-by-Step Implementation Plan
1. In `packages/shared/src/models/SessionState.ts`, define the full `SessionState` type per SCHEMA.md §3.2 (schemaVersion, metadata, notesDocument, draft, templateSelection, voiceProfileRef, formattedDocument, reviewerFeedback).
2. In `packages/shared/src/schemas/session-state.schema.ts`, define `SessionStateSchema` (zod) — validate the full serialized shape.
3. In `apps/api/src/lib/atomic-write.ts`, implement `atomicWrite(filePath: string, content: string): Promise<void>` — write to `filePath.tmp`, `fsync`, rename to `filePath`, copy old to `filePath.tmp.bak`.
4. In `apps/api/src/persistence/SessionStore.ts`, implement:
   - `save(state: SessionState, path: string): Promise<SaveResult>` — validate with `SessionStateSchema`, `atomicWrite`, return `{ saved: true, path, savedAt, sizeBytes }`.
   - `load(path: string): Promise<LoadResult>` — read file, validate with `SessionStateSchema`; if invalid, try `path.tmp.bak`; if both invalid, throw `CorruptProjectFileError`.
5. In `apps/api/src/routes/session.routes.ts`, implement `POST /api/v1/session/save`, `POST /api/v1/session/load`, `GET /api/v1/session/recent` (recent list stored in `~/.scholarforge/recent.json`).
6. In `apps/web/src/hooks/useAutosave.ts`, implement a debounced (2s) trigger that calls `POST /api/v1/session/save` whenever `session-store`'s state changes. Show "Saving..." / "Saved" / "Unsaved changes" in the header.
7. In `apps/web/src/components/session/SaveDialog.tsx` and `OpenDialog.tsx`, build the Save As and Open Project dialogs.
8. In `apps/web/src/components/session/AutosaveIndicator.tsx`, build the header indicator.
9. Deliberately corrupt a test save file (truncate it mid-write), verify `POST /api/v1/session/load` falls back to `.bak` and returns a `warning` field.
10. Run a full save → close app → reopen app → reload cycle 5 times against the real pipeline output from Day 6, confirming zero data loss.

### 📂 Folder Changes
Populate: `apps/api/src/persistence/`, `apps/api/src/persistence/schema/`, `apps/api/src/lib/atomic-write.ts`, `apps/web/src/components/session/`, `apps/web/src/hooks/useAutosave.ts`.

### 📄 Files
| File | Purpose |
|---|---|
| `packages/shared/src/models/SessionState.ts` | Full SessionState type |
| `packages/shared/src/schemas/session-state.schema.ts` | zod schema for SessionState |
| `packages/shared/src/interfaces/ISessionStore.ts` | Persistence contract |
| `apps/api/src/lib/atomic-write.ts` | temp-file-then-rename helper |
| `apps/api/src/persistence/SessionStore.ts` | Save/load/atomic-write implementation |
| `apps/api/src/routes/session.routes.ts` | POST /session/save, /session/load, GET /session/recent |
| `apps/web/src/hooks/useAutosave.ts` | Debounced autosave trigger |
| `apps/web/src/components/session/SaveDialog.tsx` | Save As dialog |
| `apps/web/src/components/session/OpenDialog.tsx` | Open Project dialog |
| `apps/web/src/components/session/AutosaveIndicator.tsx` | Header autosave status |
| `apps/api/tests/unit/persistence/SessionStore.test.ts` | Round-trip serialization tests |
| `apps/api/tests/unit/persistence/CorruptionRecovery.test.ts` | Deliberate-corruption recovery tests |

### 📦 Libraries / Packages
- Standard `fs` (Node native) — no new dependency for atomic writes.
- `zod` (already installed) — for schema validation.

### 🔗 Integrations
- `SessionStore` depends on, but does not modify, every shared model from Days 2–6.
- The frontend's Zustand `session-store` is the source of truth; autosave is frontend-triggered (the backend is stateless per ADR-008).
- `AutosaveScheduler` hooks into the Zustand store's `subscribe()` method, not by workflows calling persistence directly.

### 🧪 Testing Tasks
- **Unit:** Full round-trip save→load equality test against the Day 6 real pipeline output — every field matches after reload.
- **Unit:** Corruption recovery — truncate a save file, verify `.bak` fallback; corrupt both, verify explicit error.
- **Integration:** Autosave fires after a note edit, after draft generation, after a template change.
- **Manual:** Kill the backend process mid-autosave, confirm the last good autosave still loads.
- **Edge cases:** Loading a file with `schemaVersion: 2` (should fail with `UNSUPPORTED_SCHEMA_VERSION`, not silent parse).
- **Expected outputs:** Zero data loss across 5+ save/reload cycles (directly maps to the PRD success metric).

### 🐞 Common Bugs
- **Bug:** Autosave fires too frequently, causing UI stutter. **Root cause:** debounce window too short, or triggered on every keystroke. **Recovery:** widen debounce to 2s, trigger on discrete events (not every character).
- **Bug:** Atomic rename leaves both old file and orphaned temp. **Root cause:** rename not atomic on the target filesystem. **Recovery:** add a startup check that resolves leftover temp files (prefer the most recently completed write).
- **Bug:** Deserialized objects don't behave like freshly-generated ones. **Root cause:** raw JSON-parsed objects passed around directly. **Recovery:** reconstruct objects through proper constructors — `JSON.parse` then validate with zod, then cast to the typed interface.

### ✅ End-of-Day Checklist
- [ ] `ISessionStore`/`SessionStore` implemented with atomic writes.
- [ ] `POST /api/v1/session/save`, `/session/load`, `GET /api/v1/session/recent` endpoints working.
- [ ] `SessionStateSchema` (zod) defined and validated on every load.
- [ ] Corruption recovery flow tested against a deliberately broken file.
- [ ] Zero data loss confirmed across 5+ save/reload cycles.
- [ ] Autosave indicator in the header reflects save status.
- [ ] Save As / Open Project dialogs functional.

### ➡️ Handoff Notes
Session persistence is complete and proven robust. The full pipeline now survives app restarts with no data loss. Day 8 adds Reviewer Mode as a new, independent AI-layer component and hardens error handling across the whole pipeline.

---

## Day 8

### 🎯 Objective
Build **Reviewer Mode** (`ReviewerEngine`) end-to-end, and perform a dedicated **error-handling hardening pass** across the entire pipeline (Architecture §12).

### 📖 Learning Outcomes
- Designing a second, distinct AI pipeline (critique-framed) that reuses `IModelClient`/`OutputCleaner` but not generation templates.
- Systematic error-handling hardening as a discrete engineering activity.
- Ensuring feedback is traceable to actual draft content.

### 🛠 Features to Build
- `ReviewerFeedback`, `ReviewResult` types in `packages/shared`.
- `ReviewerEngine` in `apps/api/src/ai/reviewer/` — critique-framed prompts per section.
- Reviewer prompt template JSON files in `apps/api/src/ai/reviewer/ReviewerPromptTemplates/`.
- `POST /api/v1/generate/review` endpoint.
- Reviewer Panel UI in the frontend (`/project/[id]/review`).
- Error-handling hardening: missing-model pre-flight, export-failure recovery, corrupt-session fallback (extend Day 7), template-mismatch reporting.

### 📝 Step-by-Step Implementation Plan
1. In `packages/shared/src/models/ReviewerFeedback.ts`, define `ReviewerFeedback { sectionId, issue, suggestion, severity, category }` and `ReviewResult { feedback: ReviewerFeedback[], reviewedAt, reviewedDraftId }`.
2. In `apps/api/src/ai/reviewer/ReviewerPromptTemplates/`, create JSON files per category: `clarity.json`, `argument.json`, `evidence.json`, `style.json`, `structure.json`. Each instructs the model to identify weaknesses of that type, referencing actual section content.
3. In `apps/api/src/ai/reviewer/ReviewerEngine.ts`, implement `review(draft: Draft, config: ReviewerConfig): Promise<ReviewResult>` — iterate `draft.sections`, build a critique prompt per section (reusing `IModelClient`, not `PromptBuilder`), clean output via `OutputCleaner`, parse into `ReviewerFeedback[]`.
4. In `apps/api/src/routes/generate.routes.ts`, add `POST /api/v1/generate/review` — parse body, call `ReviewerEngine.review()`, return `ReviewResult`.
5. In `apps/web/src/app/project/[id]/review/page.tsx`, build the Reviewer Panel (see UI-WIREFRAMES.md §3.4). Add the `useGenerateReview` hook.
6. **Error-handling hardening pass**, one item at a time, each with a Vitest test:
   - Missing model: pre-flight check in `GenerationWorkflow` and `ReviewerEngine` — if `/health` reports model unreachable, return 503 immediately with `MODEL_UNREACHABLE` error code.
   - Export failure: wrap `DocxExporter` calls — on failure, report which section/element failed, preserve `FormattedDocument` for retry.
   - Corrupt session: verify Day 7's fallback-to-autosave reports a clear message if both fail.
   - Template mismatch: confirm `TemplateEngine.validate()`'s specific error messages surface to the UI, not just logs.
7. Deliberately trigger each failure mode via the UI and confirm the user-facing message is specific and actionable, not a raw stack trace.

### 📂 Folder Changes
Populate: `apps/api/src/ai/reviewer/`, `apps/api/src/ai/reviewer/ReviewerPromptTemplates/`, `apps/web/src/app/project/[id]/review/`, `apps/web/src/hooks/useGenerateReview.ts`. Add `ReviewerFeedback` to `packages/shared/src/models/`.

### 📄 Files
| File | Purpose |
|---|---|
| `packages/shared/src/models/ReviewerFeedback.ts` | Structured feedback data model |
| `apps/api/src/ai/reviewer/ReviewerEngine.ts` | Critique generation logic |
| `apps/api/src/ai/reviewer/ReviewerPromptTemplates/*.json` | Critique-framed prompt templates |
| `apps/api/src/routes/generate.routes.ts` (updated) | Adds POST /generate/review |
| `apps/web/src/app/project/[id]/review/page.tsx` | Reviewer Panel UI |
| `apps/web/src/hooks/useGenerateReview.ts` | TanStack Query mutation |
| `apps/api/tests/integration/ReviewerEngine.test.ts` | Mocked + real-model reviewer tests |
| `docs/adr/ADR-007-error-handling-pass.md` | Records each hardened failure mode + test |

### 📦 Libraries / Packages
- No new libraries — Reviewer Mode reuses Day 3's `IModelClient` and Day 4's `OutputCleaner` entirely. This is a deliberate confirmation that the architecture's layering pays off: a whole new feature (FR-8) requires zero new infrastructure.

### 🔗 Integrations
- `ReviewerEngine` depends on `IModelClient` (Day 3) and `OutputCleaner` (Day 4), consumes `Draft` (Day 4/7) — it does not depend on `TemplateEngine` or `DocxExporter` at all.
- Error-handling hardening touches existing workflows only at their public interface boundaries — no internal logic restructuring.

### 🧪 Testing Tasks
- **Unit:** `ReviewerFeedback` parsing from raw model text into structured entries, against several raw-output shapes (well-formed, malformed, empty).
- **Integration:** `ReviewerEngine.review()` against the real Day 6 draft — manually verify at least 2 pieces of feedback quote or clearly reference something specific from the actual text.
- **Manual:** Trigger each of the four hardened failure modes deliberately and confirm every one produces a specific, actionable message.
- **Edge cases:** Running Reviewer Mode on a very short section; running it twice in a row (confirm feedback isn't duplicated).

### 🐞 Common Bugs
- **Bug:** Reviewer feedback is generic ("Consider improving clarity"). **Root cause:** critique prompt didn't include enough actual section text. **Recovery:** ensure the prompt explicitly requires referencing a specific phrase/claim from the section.
- **Bug:** Specific errors in logs but generic "Something went wrong" in UI. **Root cause:** a catch-all error boundary swallows the specific error. **Recovery:** ensure error objects propagate through to the UI, not replaced by a default message.
- **Bug:** Reviewer Mode accidentally triggers full re-generation. **Root cause:** workflow wiring mistake reusing `GenerationWorkflow`. **Recovery:** verify "Run Reviewer" is wired to `ReviewerEngine`, not `GenerationWorkflow`.

### ✅ End-of-Day Checklist
- [ ] `ReviewerEngine` implemented, producing content-specific feedback.
- [ ] `POST /api/v1/generate/review` endpoint working.
- [ ] Reviewer Panel UI functional.
- [ ] All four hardened error-handling scenarios tested and confirmed.
- [ ] No raw stack traces or generic "something went wrong" messages reach the UI.

### ➡️ Handoff Notes
All PRD functional requirements (FR-1 through FR-10) now have a working implementation for IEEE. Day 9 is dedicated to adding the remaining template standards/editions as stretch scope per the PRD's depth-over-breadth allowance.

---

## Day 9

### 🎯 Objective
Extend template coverage beyond IEEE — **APA (6th, 7th), MLA (8th, 9th), Chicago (17th, 18th)** — by adding new `TemplateDefinition` JSON files and completing the `author-date` `CitationFormatter` strategy. Explicitly stretch scope: IEEE alone is the success bar.

### 📖 Learning Outcomes
- Validating that the data-driven template architecture delivers on its promise: adding six new templates should feel mechanical.
- Handling a second citation strategy family (`author-date`).
- Prioritizing under time pressure: sequencing "most valuable first."

### 🛠 Features to Build
- `apa_6.json`, `apa_7.json`, `mla_8.json`, `mla_9.json`, `chicago_17.json`, `chicago_18.json` in `apps/api/src/formatting/templates/`.
- Complete the `author-date` `CitationFormatter` strategy (stubbed Day 5).
- `footnote` strategy if Chicago requires it (verify against real Chicago rules).
- Template Picker UI update to list all available standards/editions.

### 📝 Step-by-Step Implementation Plan
1. **Priority order** (most-requested first, in case time runs out): APA 7th → APA 6th → MLA 9th → MLA 8th → Chicago 18th → Chicago 17th.
2. For each standard/edition: research the specific mechanical rules — margins, font, spacing, heading style, title-page requirements, in-text citation format, reference-list title and entry format — and author the JSON file conforming to `TemplateDefinitionSchema`.
3. Complete `CitationFormatter`'s `author-date` strategy: in-text `(Author, Year)` format, reference-list entry per the `referenceEntryFormat` string in each template file.
4. If Chicago requires footnote/endnote citation, add a `footnote` strategy to the registry — still "add a new strategy to an existing registry," not new architecture.
5. Run the full pipeline (generation → formatting → export) against the fixture for each newly added template.
6. Update the Template Picker UI to list all completed standards/editions, defaulting to IEEE if none is chosen.
7. **Stop and reassess after each template**: if by mid-day only 2–3 of the six are done, that is an acceptable outcome per the PRD. Log remaining templates as "not completed in V1" rather than shipping a rushed, untested file.

### 📂 Folder Changes
No new folders — only new files in `apps/api/src/formatting/templates/`, additions in `apps/api/src/formatting/engine/CitationFormatter.ts`.

### 📄 Files
| File | Purpose |
|---|---|
| `apps/api/src/formatting/templates/apa_7.json`, `apa_6.json` | APA template definitions |
| `apps/api/src/formatting/templates/mla_9.json`, `mla_8.json` | MLA template definitions |
| `apps/api/src/formatting/templates/chicago_18.json`, `chicago_17.json` | Chicago template definitions |
| `apps/api/src/formatting/engine/CitationFormatter.ts` (updated) | Adds `author-date` (and `footnote` if needed) |
| `apps/api/tests/unit/formatting/templates_all.test.ts` | Schema-validity + smoke test for every template file |

### 📦 Libraries / Packages
- None new — purely data authoring + completing an already-scaffolded strategy pattern.

### 🔗 Integrations
- Every new template file plugs into the exact same `TemplateEngine`/`LayoutResolver`/`DocxExporter` pipeline, unchanged. If any integration point needs to change to support a new standard, treat that as a signal the Day 5 schema had a gap — document and fix the schema (backward-compatibly), rather than special-casing.

### 🧪 Testing Tasks
- **Unit:** Schema validation for every new template file (reuse Day 3/5's validator).
- **Integration:** Full Notes→Generation→Format→Export run for each completed standard, using the same fixture.
- **Manual:** Open each exported file and visually spot-check against real APA/MLA/Chicago formatting references.
- **Expected outputs:** Every completed template produces a distinctly, correctly formatted DOCX from the identical `Draft`.

### 🐞 Common Bugs
- **Bug:** A new template "mostly works" but breaks a shared assumption. **Root cause:** Day 5's schema/resolver was implicitly built around IEEE's shape. **Recovery:** make the assumption explicit and conditional on the data (`titlePage.required: false`), not hardcoded.
- **Bug:** APA 6th and 7th are accidentally identical. **Root cause:** copy-pasting without diffing the real rule differences. **Recovery:** explicitly diff the two editions' rules before finalizing both files.
- **Bug:** Running out of time leaves a half-correct template. **Recovery:** better to ship fewer, fully-correct templates — remove an incomplete template from the registry rather than leaving it selectable and broken.

### ✅ End-of-Day Checklist
- [ ] At least one additional standard beyond IEEE fully completed and tested end-to-end (stretch: all six).
- [ ] `author-date` citation strategy implemented and tested.
- [ ] Every completed template passes schema validation and produces a valid DOCX.
- [ ] Any incomplete/untested templates explicitly removed from the registry.
- [ ] IEEE reconfirmed still working after any shared-code changes.

### ➡️ Handoff Notes
Template coverage extended as far as time allowed, with IEEE remaining the fully-proven baseline. Day 10 is the final integration, polish, testing, and demo-rehearsal day.

---

## Day 10

### 🎯 Objective
Final **integration, polish, full regression testing, and live demo rehearsal**. No new features. The goal is a stable, demoable, submittable V1.0 that satisfies every PRD acceptance criterion.

### 📖 Learning Outcomes
- Running a full acceptance-criteria checklist against a real system.
- Rehearsing a live, cold-start demo and hardening against failure modes.
- Making judgment calls about what "done" means without slipping into feature work.

### 🛠 Features to Build
None. This is explicitly a stabilization, testing, and polish day — building any new feature today is scope creep.

### 📝 Step-by-Step Implementation Plan
1. Run the full Vitest regression suite (all unit + integration tests from Days 2–9) and fix any regressions before doing anything else.
2. Walk the PRD's Acceptance Criteria (§10) line by line as a literal checklist against the real running app — not a code review, an actual click-through:
   - Notes + IEEE (and any other completed standard) → correctly formatted complete draft.
   - Voice profile produces text recognizably distinct from generic AI output (manual judgment call, documented).
   - Save → reload restores everything with no data loss.
   - Reviewer Mode returns content-specific feedback.
   - DOCX export is valid and opens cleanly.
   - **Entire workflow completes with no network connection active** — literally disconnect the network and run the full flow once, to prove NFR-1.
   - Full demo scenario runs on the actual 4GB GPU target hardware without crashing (NFR-7).
3. UI polish pass: consistent spacing/labeling across all views — no functional changes, cosmetic only, time-boxed strictly.
4. Prepare and rehearse the **exact demo script** end-to-end at least 3 times, cold (closing and reopening the app each time), timing each run.
5. Deliberately induce each of Day 8's four hardened failure modes one more time during rehearsal to confirm they still behave correctly after Day 9's changes.
6. Freeze the codebase (git tag `v1.0.0-rc1`) once the full acceptance checklist passes three consecutive cold-start rehearsals with no crash.
7. Write the `README.md`, `docs/installation_guide.md`, `docs/demo_script.md`, `docs/acceptance_checklist_results.md`, `CHANGELOG.md`.

### 📂 Folder Changes
No structural changes. Add/finalize: `README.md`, `docs/installation_guide.md`, `docs/demo_script.md`, `docs/acceptance_checklist_results.md`, `CHANGELOG.md`.

### 📄 Files
| File | Purpose |
|---|---|
| `README.md` | Primary project documentation for GitHub |
| `docs/installation_guide.md` | Step-by-step setup for a new machine |
| `docs/demo_script.md` | The exact rehearsed demo flow |
| `docs/acceptance_checklist_results.md` | Filled-in PRD §10 checklist with pass/fail evidence |
| `CHANGELOG.md` | Summary of V1.0 (includes + exclusions) |

### 📦 Libraries / Packages
None new. Today is explicitly a zero-new-dependency day — any new library introduced this late is an unacceptable stability risk.

### 🧪 Testing Tasks
- **Regression:** Full Vitest suite, run fresh, zero failures tolerated.
- **Acceptance:** Every PRD §10 acceptance criterion checked off with actual evidence (screenshot, test log, or timed observation).
- **Manual (cold-start demo rehearsal):** Minimum 3 full cold runs of the entire notes→export flow, network disconnected, on the actual target 4GB GPU hardware, no crashes.
- **Edge cases (final sweep):** Extremely long notes document (stress-test NFR-5); rapid repeated save actions; switching templates mid-session after a draft exists.
- **Expected outputs:** Three consecutive clean cold-start demo runs; a fully checked-off acceptance criteria document; zero known open crashes.

### 🐞 Common Bugs
- **Bug:** Something works in isolated testing but fails during the full cold-start sequence. **Root cause:** startup-ordering assumption (config loaded before model daemon confirmed). **Recovery:** add an explicit startup sequence check.
- **Bug:** Demo works at a desk but stutters in front of an audience due to GPU thermal throttling. **Root cause:** environmental. **Recovery:** rehearse in conditions close to the real demo environment; close background applications.
- **Bug:** A "polish" UI tweak accidentally breaks a data-binding. **Root cause:** cosmetic changes touching functional code without re-testing. **Recovery:** re-run the relevant test suite after every polish change.

### ✅ End-of-Day Checklist
- [ ] Full Vitest regression suite green.
- [ ] Every PRD §10 acceptance criterion checked off with evidence.
- [ ] Three consecutive clean cold-start demo rehearsals on target hardware.
- [ ] README, installation guide, and demo script written.
- [ ] Release candidate tagged/frozen (`v1.0.0-rc1`).
- [ ] No new features or dependencies introduced today.

### ➡️ Handoff Notes
Version 1.0 is complete, tested against every PRD acceptance criterion, and demo-rehearsed cold on the actual target hardware. Any future work belongs to Version 2.0 planning, using the extension points already architected in — none of which require reopening or refactoring any V1 code.

---

## Deployment Checklist

- [ ] **Repository cleanup:** remove debug harnesses/scratch scripts (or mark them clearly in `tools/`); remove `.gitkeep` files that now have real content; confirm `.gitignore` excludes local model files, session project files, and build artifacts.
- [ ] **README:** project description, tagline, screenshot, quick-start, link to installation guide, explicit V1 scope + exclusions (copied faithfully from Vision §6/§7).
- [ ] **Installation guide:** OS prerequisites, GPU/driver requirements (4GB minimum), Ollama setup, Qwen2.5-3B model download, application install/run steps.
- [ ] **Requirements:** exact dependency list with versions (Node 20 LTS, Next.js 14+, Express 4, docx, zod, vitest), hardware minimums.
- [ ] **Environment setup:** step-by-step from a clean machine to a running app, validated by literally following it on a fresh environment.
- [ ] **Sample project:** ship the Day 2 fixture as an example `.scholarforge` project new users can open immediately.
- [ ] **Demo workflow:** the exact rehearsed Day 10 demo script, written clearly enough that someone other than the developer could follow it.
- [ ] **Testing checklist:** the full Vitest suite (unit + integration across all 9 days) is documented and runnable via `pnpm test`.
- [ ] **Documentation checklist:** Vision, PRD, Architecture, Blueprint, and Day 52 deliverables all included in `docs/` alongside the README.
- [ ] **Performance checklist:** documented, empirically-measured generation latency per section and total pipeline time on the target 4GB GPU (from ADR-003 + Day 10 stress testing).
- [ ] **Privacy checklist:** explicit statement (and architectural confirmation per Architecture §13) that no network calls occur outside localhost during normal operation; confirmed by the Day 10 network-disconnected test run.
- [ ] **Release checklist:** version tag applied, CHANGELOG finalized, release candidate branch frozen, final regression suite green.

---

## Milestone Tracking Table

| Day | Objective | Key Deliverables | Dependencies | Status Criteria |
|---|---|---|---|---|
| 2 | Monorepo skeleton + Notes Module | pnpm workspace, `packages/shared`, Notes Editor UI, `NotesValidator`, `POST /notes/validate` | Day 1 environment | Sample fixture validates cleanly; Vitest tests green |
| 3 | Configuration + AI connectivity | `ConfigurationService`, `IModelClient`, `OllamaClient`/`LlamaCppClient`, `GET /health`, `GET /config/model` | Day 2 | Real model responds within 4GB budget via debug harness |
| 4 | Prose generation pipeline | `PromptBuilder`, `VoiceProfile`, `OutputCleaner`, `GenerationWorkflow`, `POST /generate/draft` (+stream), Draft View | Days 2–3 | Full `Draft` generated from sample notes, voice-profile influence visible |
| 5 | Deterministic Template Engine + IEEE | `TemplateEngine`, `LayoutResolver`, `CitationFormatter`, `ieee_us_letter.json`, `POST /format/apply` | Day 4 | `FormattedDocument` for IEEE resolves correctly, Vitest suite passes without model |
| 6 | DOCX Export | `DocxExporter`, `DocxRenderer`, `StyleMapper`, `POST /export/docx`, Export View | Day 5 | Valid `.docx` opens cleanly in Word/LibreOffice |
| 7 | Session persistence | `SessionStore`, `useAutosave` hook, `POST /session/*` endpoints, Save/Open dialogs | Days 2–6 | Zero data loss across 5+ save/reload cycles; corruption recovery verified |
| 8 | Reviewer Mode + error hardening | `ReviewerEngine`, `POST /generate/review`, Reviewer Panel, four hardened error handlers | Days 3–4, Day 7 | Content-specific feedback; all four failure modes gracefully handled |
| 9 | Additional templates (stretch) | APA/MLA/Chicago JSON files, `author-date` strategy | Day 5 | At least one additional standard fully working; IEEE reconfirmed |
| 10 | Integration, testing, demo rehearsal | Full regression pass, acceptance checklist, README/docs, 3 clean cold-start demos | All prior days | Three crash-free cold-start demos; every PRD §10 criterion checked |

---

## Quality Gates

Each day's work must clear its gate before the next day begins.

| Gate (end of Day) | Must be true before proceeding |
|---|---|
| 2 | Sample notes fixture validates with zero issues; `POST /notes/validate` returns 200; Vitest unit tests all pass |
| 3 | Real local model returns a real response within the 4GB VRAM budget via the debug harness; `GET /health` reports model reachability |
| 4 | A complete, readable `Draft` is generated end-to-end from the sample fixture; Draft View UI shows progress + output |
| 5 | `FormattedDocument` for IEEE resolves correctly; Vitest Formatting Layer suite passes without the model running |
| 6 | Exported `.docx` opens without a repair prompt in a real word processor, matching IEEE layout expectations |
| 7 | Zero data loss confirmed across 5+ save/reload cycles, including one deliberate-corruption recovery test |
| 8 | `ReviewerEngine` produces content-specific feedback; all four hardened error scenarios behave correctly |
| 9 | At least one additional standard beyond IEEE is fully working end-to-end; IEEE still passes regression |
| 10 | Three consecutive crash-free cold-start demo runs on target hardware; every PRD §10 acceptance criterion checked with evidence |

---

## References

| Document | Location |
|---|---|
| ScholarForge V1.0 Product Vision (Approved) | `docs/product_vision.md` |
| ScholarForge PRD (Draft for Build) | `docs/prd.md` |
| ScholarForge Software Architecture Document (V1.0) | `docs/architecture.md` |
| Original Implementation Blueprint (Days 2–10) | `docs/implementation_blueprint.md` |
| **This updated blueprint** | `Day52/IMPLEMENTATION-BLUEPRINT.md` |
| Day 52 Architecture | `Day52/ARCHITECTURE.md` |
| Day 52 Storage Architecture | `Day52/SCHEMA.md` |
| Day 52 API Specification | `Day52/API.md` |
| Day 52 Project Structure | `Day52/PROJECT-STRUCTURE.md` |
| Day 52 UI Wireframes | `Day52/UI-WIREFRAMES.md` |
| Day 52 Readiness Review | `Day52/day52.md` |
