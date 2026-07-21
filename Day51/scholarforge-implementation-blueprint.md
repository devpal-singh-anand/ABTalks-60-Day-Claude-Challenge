# ScholarForge — Engineering Implementation Blueprint (Days 2–10)

**Source of truth:** ScholarForge V1.0 Product Vision (Approved) + PRD (Draft for Build) + Software Architecture Document (V1.0)
**Author role:** Senior TPM / Engineering Manager / Agile Sprint Planner
**Scope discipline:** No feature added, removed, or reinterpreted. This document sequences the *already-approved* scope into an executable day-by-day plan for a solo developer. Day 1 (kickoff/environment groundwork) is already complete; this blueprint begins at Day 2.

**Sequencing logic:** Each day builds one vertical slice of the architecture in dependency order — Notes → Config/AI plumbing → Generation → Deterministic Formatting (IEEE first, per the PRD's explicit depth-over-breadth assumption) → Export → Persistence → Reviewer Mode → Additional templates (stretch) → Integration/Polish/Demo. This order front-loads the riskiest unknowns (model behavior on 4GB VRAM, deterministic/AI seam) early, and pushes the safest, most mechanical work (additional template editions) to the day most likely to be compressed if schedule slips.

---

## Day 2

### 🎯 Objective
Stand up the project skeleton exactly as defined in the Architecture Document's folder structure, and build the **Notes Module** (structured research notes editor + validation) — the first functional vertical slice, with zero AI or formatting dependencies yet.

### 📖 Learning Outcomes
- Structuring a modular local application around layer boundaries (Presentation / Application / shared data models) before any "interesting" logic exists.
- Designing a normalized structured-data schema (`NotesDocument`) that will need to survive unchanged through AI, Formatting, and Export layers later.
- Writing a validator that fails fast and specifically, not silently.

### 🛠 Features to Build
- Full repository skeleton (all folders from the Architecture Document §4, even empty ones with `.gitkeep` placeholders).
- `NotesDocument` / `NoteSection` shared data model.
- Notes Editor view: add/edit/remove sections, each with a heading, bullet points, and optional source references.
- `NotesValidator`: checks every section has a heading and at least one bullet before generation is allowed downstream.

### 📝 Step-by-Step Implementation Plan
1. Create the full folder tree from the Architecture Document §4 (`app/`, `application/`, `ai/`, `formatting/`, `persistence/`, `export/`, `config/`, `shared/`, `tests/`, `docs/`). Empty folders get a placeholder file so they persist in version control.
2. Initialize the project (language/runtime of choice — pick once here and do not revisit; this decision is out of scope for this blueprint since the Architecture Document deliberately stayed implementation-language-agnostic).
3. In `shared/models/`, define `NotesDocument { sections: NoteSection[] }` and `NoteSection { id, heading, bullets: string[], sourceRefs: SourceRef[] }`, and `SourceRef { author, year, title, source }` (minimal fields needed later for citation formatting — do not over-design this; only what the PRD's citation requirement needs).
4. In `application/validation/`, implement `NotesValidator.validate(doc): ValidationResult` returning a list of specific issues (e.g., "Section 2 has no bullets") rather than a boolean.
5. In `app/views/NotesEditorView`, build a minimal UI: add section, edit heading, add/remove bullets, add source reference fields. No styling polish needed yet — function over form today.
6. Wire the view to an in-memory `NotesModule` state holder (no persistence yet — that's Day 7).
7. Add a "Validate" action in the UI that runs `NotesValidator` and displays issues inline.
8. Write a fixture: one complete, valid sample `NotesDocument` in `tests/fixtures/sample_notes/` — this fixture will be reused by every subsequent day's tests, so make it realistic (3–4 sections, at least 2 source references).

### 📂 Folder Changes
Create: `app/`, `app/views/`, `app/viewmodels/`, `application/`, `application/workflows/`, `application/validation/`, `ai/` (empty scaffolding only), `formatting/` (empty scaffolding only), `persistence/` (empty scaffolding only), `export/` (empty scaffolding only), `config/`, `shared/`, `shared/models/`, `shared/interfaces/`, `tests/`, `tests/unit/`, `tests/fixtures/sample_notes/`, `docs/`, `docs/adr/`.

### 📄 Files
| File | Purpose |
|---|---|
| `shared/models/NotesDocument.*` | Core data model for structured notes |
| `shared/models/NoteSection.*` | Individual section schema |
| `shared/models/SourceRef.*` | Minimal citation-source schema, reused by Formatting layer later |
| `application/validation/NotesValidator.*` | Structural validation logic |
| `app/views/NotesEditorView.*` | UI for entering/editing notes |
| `app/viewmodels/NotesEditorViewModel.*` | Thin adapter between view and validator/state |
| `tests/fixtures/sample_notes/valid_sample.json` | Canonical test fixture used across all later days |
| `tests/unit/validation/NotesValidator.test.*` | Unit tests for validator edge cases |
| `docs/adr/ADR-000-day2-kickoff.md` | Record of language/runtime/project-init decisions made today |

### 📦 Libraries / Packages
- UI framework of choice for the target platform (desktop-local app) — install per its standard setup; no other day depends on which one, only that it exists.
- A test runner appropriate to the chosen language (e.g., a standard unit-test framework) — needed starting today since NotesValidator gets tests immediately.
- No AI, DOCX, or persistence libraries yet — deliberately deferred to the days that need them, keeping today's dependency surface minimal.

### 🔗 Integrations
- None yet — today's module has no dependency on AI, Formatting, Persistence, or Export layers. This is intentional: the Notes Module must be provably correct and stable before anything is built on top of it.
- `shared/models/NotesDocument` becomes the contract every later layer consumes — treat changes to this schema as expensive starting tomorrow.

### 🧪 Testing Tasks
- **Unit:** `NotesValidator` against: empty document, section with no heading, section with no bullets, valid document, document with duplicate section IDs.
- **Manual:** Add 3 sections through the UI, remove one, edit a heading, verify state reflects correctly.
- **Edge cases:** Very long bullet text, special characters in headings, a section with 0 source references (must be allowed — sources are optional per PRD, only structure is required).
- **Expected outputs:** Validator returns an empty issue list for the fixture file; returns specific, human-readable issues for each broken case above.

### 🐞 Common Bugs
- **Bug:** UI state and validator disagree because the viewmodel copies data instead of referencing the single source of truth. **Root cause:** premature optimization/caching. **Debug:** log the object identity, not just contents, at each layer. **Recovery:** always pass the same `NotesDocument` reference through the pipeline; only copy at explicit save points.
- **Bug:** Validator passes but downstream (future days) chokes on an empty `bullets` array that slipped through. **Root cause:** validator logic checked for the array's existence, not its length. **Recovery:** add an explicit length check test today, not later.

### 📸 Expected Screenshots
1. Empty Notes Editor on first launch.
2. Notes Editor with the full sample fixture entered (3–4 sections visible).
3. Validation error state (intentionally break a section, show the specific error message).
4. Folder tree of the full project skeleton (terminal or IDE sidebar view) for documentation.

### ✅ End-of-Day Checklist
- [ ] Full folder skeleton exists and matches Architecture Document §4.
- [ ] `NotesDocument`/`NoteSection`/`SourceRef` models implemented and covered by at least one test each.
- [ ] `NotesValidator` implemented with specific, non-generic error messages.
- [ ] Notes Editor UI functional for add/edit/remove of sections and bullets.
- [ ] Sample fixture (`valid_sample.json`) created and passes validation.
- [ ] All Day 2 unit tests green.
- [ ] Screenshots captured and saved to `docs/`.

### ➡️ Handoff Notes
The project skeleton exists and matches the approved architecture exactly. The Notes Module is complete, tested, and produces a validated `NotesDocument` — this is the only artifact Day 3 needs as an input assumption. **Day 3 does not touch the Notes Editor at all**; it starts the Configuration Layer and AI Layer scaffolding (interfaces + Ollama/llama.cpp client stubs) completely independently, and will consume `tests/fixtures/sample_notes/valid_sample.json` as its test input rather than requiring the UI to be driven manually. No AI, formatting, or persistence code exists yet — this is expected and correct at this point in the sequence.

---

## Day 3

### 🎯 Objective
Build the **Configuration Layer** in full, and the **AI Layer's connectivity skeleton**: `IModelClient` interface plus both `OllamaClient` and `LlamaCppClient` implementations, validated against the real local model with a trivial hardcoded prompt (no prompt builder yet — that's Day 4). By end of day, the riskiest unknown in the whole project (does Qwen2.5-3B run acceptably on this 4GB GPU) must be empirically answered.

### 📖 Learning Outcomes
- Designing a backend-agnostic adapter interface (`IModelClient`) so business logic never knows which local runtime is active.
- Practical quantization/context-window tuning for a 3B model under a 4GB VRAM ceiling.
- Reading and validating configuration schemas defensively (config errors must fail loudly at startup, not mid-generation).

### 🛠 Features to Build
- `ConfigurationService` with typed accessors and startup validation.
- `model.config.json` schema (backend choice, model path/name, quantization, context window, temperature, timeout).
- `IModelClient` interface.
- `OllamaClient` and `LlamaCppClient` concrete implementations (raw prompt in, raw text out — no cleaning or prompt engineering yet).
- A throwaway CLI/debug harness to send a fixed test prompt and print the raw response + latency + memory usage.

### 📝 Step-by-Step Implementation Plan
1. Install and verify the chosen local backend (Ollama daemon running locally, or llama.cpp built/binaries in place) completely independent of the application — confirm you can manually query it before writing any adapter code.
2. Pull/load Qwen2.5-3B in the chosen backend; test a manual prompt outside the app first to get a real baseline for latency and memory footprint on this specific 4GB GPU.
3. Define `model.config.json` schema in `config/`: `{ backend: "ollama"|"llamacpp", modelName, quantization, contextWindow, temperature, topP, timeoutMs }`.
4. Implement `ConfigurationService.load()` in `config/` — reads all config files at startup, validates required fields, throws a specific, readable error if anything is missing or malformed (this is the pattern every later config file should follow).
5. In `shared/interfaces/`, define `IModelClient { generate(prompt: string, options): Promise<string> }`.
6. In `ai/client/`, implement `OllamaClient` (talks to local Ollama HTTP API on localhost only) and `LlamaCppClient` (talks to local llama.cpp server/binary) — both conforming to `IModelClient`.
7. Add a factory/selector: `ModelClientFactory.create(config): IModelClient`, chosen purely from `model.config.json`'s `backend` field — no other code should ever branch on backend type.
8. Build a minimal debug harness (script or hidden dev-only UI action) that: loads config → creates client via factory → sends one fixed test prompt → logs raw response, response time, and (if obtainable) VRAM usage.
9. Empirically test 2–3 quantization/context-window combinations against the 4GB ceiling; record results in `docs/adr/` as a decision record (this feeds directly into ADR-003 from the Architecture Document, now made concrete with real numbers).
10. Lock in the final `model.config.json` defaults based on this testing — this becomes the config every later day assumes works.

### 📂 Folder Changes
Populate: `config/`, `ai/client/`, `shared/interfaces/`. No new top-level folders needed (all existed as scaffolding since Day 2).

### 📄 Files
| File | Purpose |
|---|---|
| `config/ConfigurationService.*` | Loads/validates all config, exposes typed getters |
| `config/model.config.json` | Backend, model, quantization, context, timeout settings |
| `shared/interfaces/IModelClient.*` | Contract every backend adapter must satisfy |
| `ai/client/OllamaClient.*` | Ollama-backed implementation |
| `ai/client/LlamaCppClient.*` | llama.cpp-backed implementation |
| `ai/client/ModelClientFactory.*` | Selects implementation based on config |
| `tools/debug_model_harness.*` (or equivalent dev-only entry point) | Manual smoke-test tool for latency/memory validation |
| `docs/adr/ADR-003-model-backend-tuning.md` | Records actual empirical quantization/context findings for this hardware |

### 📦 Libraries / Packages
- HTTP client library (for `OllamaClient`, talking to `localhost`) — standard library is usually sufficient, avoid heavy dependencies.
- Process/IPC utility if `LlamaCppClient` shells out to a local binary rather than a server — use the language's standard process-spawning facility.
- JSON schema validation utility for `ConfigurationService` (lightweight; avoid pulling in a large validation framework for a small config file).
- **Installation notes:** Ollama must be installed and running as a local service before `OllamaClient` can be tested; llama.cpp must be built/obtained separately if that backend is exercised. Neither requires any paid account or API key (NFR-6).

### 🔗 Integrations
- `ConfigurationService` becomes a dependency of every layer from here forward — no module should read a config file directly except through it.
- `IModelClient` is the seam the entire rest of the AI Layer (Day 4's `PromptBuilder`/`GenerationWorkflow`) will depend on — today's job is to make this seam boring and reliable so tomorrow's prompt-engineering work isn't fighting connectivity issues too.

### 🧪 Testing Tasks
- **Unit:** `ConfigurationService` against a valid config file, a missing-field config file, a malformed-JSON config file.
- **Integration:** Real call through `ModelClientFactory` → chosen `IModelClient` → real local model, with the fixed test prompt, confirming a non-empty response returns within the configured timeout.
- **Manual:** Switch `backend` in `model.config.json` between `ollama` and `llamacpp` (if both are set up) and confirm the harness works identically either way with no code changes.
- **Edge cases:** Model not running (Ollama daemon stopped) — confirm the client throws a specific, catchable error rather than hanging indefinitely; timeout value actually enforced.
- **Expected outputs:** Harness prints a coherent (even if unrefined) text response, confirms VRAM stayed within the 4GB budget, and reports response latency in an acceptable range (target: single-digit seconds for a short prompt).

### 🐞 Common Bugs
- **Bug:** Ollama/llama.cpp silently OOMs or hangs on too-large a context window. **Root cause:** context window set higher than the 4GB budget allows for this quantization level. **Debug:** monitor GPU memory during the harness run; reduce context window incrementally until stable. **Recovery:** record the safe ceiling in the config defaults and ADR-003.
- **Bug:** `OllamaClient` works from a manual `curl`/HTTP test but not from the app. **Root cause:** usually a base-URL or port mismatch, or the daemon not started before the app runs. **Recovery:** add a startup pre-flight check (Day 8 will formalize this into full "missing model" error handling; today just note it in the ADR).
- **Bug:** Timeout logic doesn't actually cancel the underlying request, leaving zombie processes/connections. **Root cause:** timeout implemented as a race against a promise without actually aborting the underlying call. **Recovery:** use the HTTP/process library's native abort/cancel mechanism, not a bare timer race.

### 📸 Expected Screenshots
1. Terminal output showing Ollama (or llama.cpp) successfully loading Qwen2.5-3B.
2. Debug harness output: prompt in, raw response out, with latency/memory numbers visible.
3. GPU memory monitor (e.g., `nvidia-smi` or platform equivalent) during a generation call, showing it stays under the 4GB ceiling.

### ✅ End-of-Day Checklist
- [ ] `ConfigurationService` implemented, validated, tested.
- [ ] `IModelClient` interface defined and stable.
- [ ] Both `OllamaClient` and `LlamaCppClient` implemented (at minimum, the one you intend to demo with must be fully working; the other may be thinner but must compile/exist per the "either backend" requirement).
- [ ] Empirical quantization/context testing completed and documented in ADR-003.
- [ ] Debug harness successfully generates a real response from the real local model within the 4GB budget.
- [ ] Final `model.config.json` defaults locked in.

### ➡️ Handoff Notes
The model connectivity seam is proven end-to-end: config loads, the correct backend is selected, and a real prompt gets a real response from Qwen2.5-3B within the 4GB hardware budget, with empirically-tuned quantization/context settings recorded in ADR-003. **Day 4 builds directly on top of `IModelClient`** — it does not touch `OllamaClient`/`LlamaCppClient` internals again unless a bug surfaces; it adds the `PromptBuilder`, `VoiceProfile`, `OutputCleaner`, and `GenerationWorkflow` on top of this now-stable foundation, using the same fixed test prompt harness as a sanity check before moving to real section-based prompts.

---

## Day 4

### 🎯 Objective
Build the full **prose-generation pipeline**: `PromptBuilder`, `VoiceProfile`, `OutputCleaner`, and `GenerationWorkflow`, producing a real end-to-end `Draft` (ordered cleaned sections) from the Day 2 `NotesDocument` fixture, using the Day 3 `IModelClient`. This is the first day the "notes → generated academic prose" core value proposition becomes real, even though it isn't formatted or exported yet.

### 📖 Learning Outcomes
- Prompt engineering for small (3B-class) models: keeping per-call context small and instruction-dense rather than relying on long few-shot chains.
- Designing a voice/style injection mechanism that's honest about being approximate (per the Architecture Document's explicit stance on this).
- Building resilient generation loops: retry-on-empty, partial-failure tolerance across sections.

### 🛠 Features to Build
- `VoiceProfile` data model + one default profile file.
- `PromptBuilder`: composes one prompt per section type, injecting note content + voice profile + section instructions.
- `OutputCleaner`: strips artifacts, validates non-empty/minimum length, triggers retry on failure.
- `GenerationWorkflow`: orchestrates the notes → per-section generate → clean → assembled `Draft` loop.
- `Draft`/`DraftSection` shared data models.

### 📝 Step-by-Step Implementation Plan
1. In `shared/models/`, define `Draft { sections: DraftSection[] }` and `DraftSection { id, sourceSectionId, heading, content }`.
2. In `ai/voice/`, define `VoiceProfile { formality, sentenceLengthTendency, personPreference, hedgingLevel, exemplarSnippets: string[] }` and create `voice_profile.default.json` in `config/` with reasonable academic-writing defaults.
3. In `ai/prompt/SectionPromptTemplates/`, create one instructional template per section type your sample notes use (e.g., Introduction, Literature Review/Background, Methodology, Discussion, Conclusion) — plain text/data files with placeholders for note content and voice-profile fields.
4. In `ai/prompt/`, implement `PromptBuilder.build(section: NoteSection, voiceProfile, sectionTemplate): Prompt` — composes the final prompt string, and enforces a max length/token budget appropriate to the context window locked in on Day 3.
5. In `ai/cleaning/`, implement `OutputCleaner.clean(rawText): string` — strip markdown artifacts/repeated boilerplate, repair or drop an incomplete trailing sentence at truncation, and expose `isAcceptable(cleanedText): boolean` (non-empty, above a minimum length threshold).
6. In `application/workflows/`, implement `GenerationWorkflow.generateDraft(notes: NotesDocument, voiceProfile, modelClient): Draft`:
   - Iterate `NotesDocument.sections` in order.
   - For each: build prompt → call `IModelClient.generate()` → clean output → if unacceptable, retry once with a slightly adjusted prompt (e.g., an explicit "write at least N sentences" instruction) → assemble into `DraftSection`.
   - Never let one section's failure abort the whole draft — log/flag the failed section and continue, returning a `Draft` with as many sections completed as possible plus a list of failed section IDs.
7. Wire a simple "Generate Draft" action into the Notes Editor UI (or a temporary dedicated Draft view) that calls `GenerationWorkflow` against the currently loaded `NotesDocument` and displays the resulting raw `Draft` sections as plain text (no formatting yet — that's Day 5).
8. Run the full pipeline against the Day 2 sample fixture and manually review output quality; adjust prompt templates iteratively today based on real output, not assumptions.

### 📂 Folder Changes
Populate: `ai/prompt/`, `ai/prompt/SectionPromptTemplates/`, `ai/voice/`, `ai/cleaning/`, `application/workflows/`. Add `Draft`/`DraftSection` to `shared/models/`.

### 📄 Files
| File | Purpose |
|---|---|
| `shared/models/Draft.*`, `shared/models/DraftSection.*` | Generated-content data model |
| `ai/voice/VoiceProfile.*` | Style parameter schema |
| `config/voice_profile.default.json` | Default style configuration |
| `ai/prompt/SectionPromptTemplates/*.json` (one per section type) | Data-driven instructional prompts |
| `ai/prompt/PromptBuilder.*` | Prompt composition logic |
| `ai/cleaning/OutputCleaner.*` | Post-processing/validation of raw model output |
| `application/workflows/GenerationWorkflow.*` | Orchestrates the full notes→draft use case |
| `app/views/DraftView.*` (minimal, plain-text for now) | Displays generated sections |
| `tests/unit/ai/OutputCleaner.test.*` | Cleaner edge-case tests |
| `tests/integration/GenerationWorkflow.test.*` | Mocked-`IModelClient` orchestration test |

### 📦 Libraries / Packages
- No new external libraries required beyond Day 3's — this is primarily business logic over the existing model client. Avoid adding an NLP/tokenization library unless the context-budget enforcement in `PromptBuilder` genuinely needs precise token counting rather than a conservative character-length heuristic (prefer the heuristic first; only add a tokenizer if testing shows it's necessary).

### 🔗 Integrations
- `GenerationWorkflow` depends on `IModelClient` (Day 3, unchanged), `NotesValidator`'s already-validated `NotesDocument` (Day 2, unchanged), and `ConfigurationService` for voice profile/model settings.
- **Critically:** `Draft` produced today has no knowledge of template/formatting concerns — it is pure content. This is the seam the Formatting Layer (Day 5) will consume without any changes needed here.

### 🧪 Testing Tasks
- **Unit:** `OutputCleaner` against: clean well-formed text, text with markdown artifacts, text truncated mid-sentence, empty string, whitespace-only string.
- **Integration:** `GenerationWorkflow` against a **mocked** `IModelClient` (deterministic fixed responses) to verify orchestration (retry logic, partial-failure handling) without needing the real model — fast, repeatable.
- **Manual (real model):** Run the full pipeline against the sample fixture with the real local model; read every generated section for coherence and voice fit.
- **Edge cases:** A section with very sparse notes (one bullet) — does the model still produce something usable, or does it need a minimum-input check added to `NotesValidator` retroactively (if so, this is a valid Day 4 fix even though `NotesValidator` was built Day 2 — flag it in the handoff, don't silently patch without documenting).
- **Expected outputs:** A complete `Draft` object with one non-empty `DraftSection` per `NoteSection`, readable prose that recognizably reflects the configured voice profile's formality level.

### 🐞 Common Bugs
- **Bug:** Generated prose ignores the voice profile entirely. **Root cause:** voice instructions buried too deep in a long prompt, or the model's small size deprioritizing subtle stylistic instructions. **Debug:** move voice instructions to the very start of the prompt; strengthen with 1–2 concrete exemplar sentences rather than abstract adjectives. **Recovery:** iterate the prompt template, not the architecture.
- **Bug:** `OutputCleaner` rejects perfectly good output because the minimum-length threshold is miscalibrated. **Root cause:** hardcoded arbitrary threshold. **Recovery:** calibrate threshold empirically today against real section lengths, store as a config value, not a magic number in code.
- **Bug:** Retries silently double VRAM pressure by not releasing the prior failed call's resources. **Root cause:** backend client not properly awaiting/cleaning up before retrying. **Recovery:** ensure retry always fully completes/cancels the prior call before issuing a new one.

### 📸 Expected Screenshots
1. Draft view showing full generated content for the sample fixture, section by section.
2. A "before cleaning / after cleaning" comparison of one raw model response (for documentation of the OutputCleaner's value).
3. Voice profile config file contents alongside a generated sample paragraph, for documentation purposes.

### ✅ End-of-Day Checklist
- [ ] `VoiceProfile` model + default config implemented.
- [ ] `PromptBuilder` producing well-formed, context-budget-respecting prompts per section.
- [ ] `OutputCleaner` implemented and unit-tested against all listed edge cases.
- [ ] `GenerationWorkflow` orchestrates full notes→draft generation with partial-failure tolerance.
- [ ] Real end-to-end run against the sample fixture produces a complete, readable `Draft`.
- [ ] Voice profile influence is subjectively recognizable in output (manual review — acceptable per the Architecture Document's stance that this can't be automated).

### ➡️ Handoff Notes
The core generative pipeline is real and working: notes go in, a cleaned, section-by-section `Draft` comes out, using the real local Qwen2.5-3B model within the 4GB budget. No formatting or export exists yet — the `Draft` is still displayed as plain, unstyled text. **Day 5 consumes this exact `Draft` object unchanged** and builds the `TemplateEngine` + the first `TemplateDefinition` (IEEE) to turn it into a `FormattedDocument`. If any prompt-quality issues remain unresolved today, document them explicitly in the handoff rather than silently carrying them forward — Day 8 (Reviewer Mode + error handling) is the designated day for deeper AI-quality hardening, not Day 5.

---

## Day 5

### 🎯 Objective
Build the generic, data-driven **Template Engine** and ship the **first complete `TemplateDefinition`: IEEE (US Letter)** — per the PRD's explicit "depth over breadth" allowance, this is the one standard that must work perfectly end-to-end before any others are attempted. By end of day, a `Draft` becomes a fully-resolved `FormattedDocument` with zero LLM involvement in any formatting decision.

### 📖 Learning Outcomes
- Building a rules engine driven entirely by data (JSON) rather than per-case code branches.
- Designing a citation-formatting strategy pattern that generalizes across in-text styles (numeric vs. author-date) without needing a subclass per standard.
- Enforcing the deterministic/AI seam in practice, not just on paper: verifying the Formatting Layer's tests never require a model to be running.

### 🛠 Features to Build
- `TemplateDefinition` shared schema + `ieee_us_letter.json` data file.
- `TemplateEngine.apply(draft, selection): FormattedDocument`.
- `CitationFormatter` (numeric strategy, for IEEE).
- `LayoutResolver` (margins, headings, spacing, page size resolution from the template data).
- `FormattedDocument` shared data model.

### 📝 Step-by-Step Implementation Plan
1. In `shared/models/`, define `FormattedDocument { sections: FormattedSection[], styleSheet: StyleSheet, references: FormattedReference[] }`, and `StyleSheet { margins, font, spacing, headings, pageSize }`.
2. In `formatting/templates/`, author `ieee_us_letter.json` conforming to the schema shown in the Architecture Document §8 (margins, font, spacing, heading levels, citation config: `inTextStyle: "numeric"`, reference format string).
3. In `formatting/engine/`, define `ITemplateEngine.apply(draft, selection): FormattedDocument` and implement `TemplateEngine` — loads the correct `TemplateDefinition` via `ConfigurationService`'s template registry, resolves layout via `LayoutResolver`, resolves citations via `CitationFormatter`, and assembles the `FormattedDocument`.
4. In `formatting/engine/`, implement `LayoutResolver` — a pure function/class translating `StyleSheet` data + `Draft` section structure into concrete layout instructions (heading level assignment per section type, paragraph indent rules, etc.) with **no conditional logic keyed on "if standard == IEEE"** — everything comes from the data file.
5. In `formatting/engine/`, implement `CitationFormatter` with a small strategy registry (`numeric`, `author-date` — only `numeric` needs to be functional today for IEEE; `author-date` can be stubbed for APA/MLA/Chicago, to be completed on Day 9).
6. Implement `ITemplateEngine.validate(document): ValidationResult` — checks the `Draft` has everything the selected `TemplateDefinition` requires (e.g., IEEE's title-page fields) before formatting, surfacing a specific "template mismatch" error rather than producing a broken document.
7. Wire a "Select Template" + "Apply Formatting" action into the UI (temporary/minimal is fine) that runs `TemplateEngine.apply()` against the Day 4 `Draft` and IEEE selection, displaying the resulting structural tree (even as raw JSON/debug view is acceptable today — visual DOCX rendering is Day 6).
8. Write the full unit-test suite for `TemplateEngine`/`LayoutResolver`/`CitationFormatter` — this is the layer the Architecture Document flags as needing the strongest test coverage, and critically, **none of these tests should require the model to be running**.

### 📂 Folder Changes
Populate: `formatting/engine/`, `formatting/templates/`. Add `FormattedDocument`/`StyleSheet`/`FormattedReference` to `shared/models/`.

### 📄 Files
| File | Purpose |
|---|---|
| `shared/models/FormattedDocument.*`, `StyleSheet.*` | Structural output of the Formatting Layer |
| `formatting/templates/ieee_us_letter.json` | First complete template definition |
| `formatting/engine/ITemplateEngine.*` | Contract for the formatting engine |
| `formatting/engine/TemplateEngine.*` | Generic engine implementation |
| `formatting/engine/LayoutResolver.*` | Data-driven layout resolution |
| `formatting/engine/CitationFormatter.*` | Strategy-based citation formatting |
| `tests/unit/formatting/TemplateEngine.test.*` | Core deterministic-layer test suite |
| `tests/unit/formatting/CitationFormatter.test.*` | Citation strategy tests |

### 📦 Libraries / Packages
- JSON schema validation (reuse whatever was chosen Day 3 for config validation — `TemplateDefinition` files should be validated the same way `model.config.json` is).
- No AI or DOCX libraries needed today — this is the point of the architecture: this entire layer is pure, dependency-light logic.

### 🔗 Integrations
- `TemplateEngine` depends on `ConfigurationService`'s template registry (a simple lookup: standard+edition → file path) and consumes `Draft` objects from Day 4 unchanged.
- Output (`FormattedDocument`) is the exact input Day 6's `DocxExporter` will consume — today's job is to make that object complete and self-sufficient (the exporter should need to make zero formatting decisions of its own tomorrow).

### 🧪 Testing Tasks
- **Unit:** `LayoutResolver` against the IEEE definition — verify margins/font/spacing/heading levels resolve exactly as specified in the data file. `CitationFormatter` numeric strategy against 2–3 sample `SourceRef` entries from the fixture.
- **Integration:** Full `TemplateEngine.apply()` against the Day 4 real generated `Draft` + IEEE selection, verifying a complete `FormattedDocument` with no missing sections.
- **Edge cases:** A `Draft` missing a section IEEE's template expects (verify `validate()` catches it with a specific message, not a crash); a `SourceRef` missing a field the citation format string needs (verify a graceful fallback or explicit error, not a silent blank).
- **Expected outputs:** A `FormattedDocument` whose `styleSheet` exactly matches `ieee_us_letter.json`'s values, and whose `references` list is correctly numbered/ordered per IEEE's numeric in-text style.

### 🐞 Common Bugs
- **Bug:** `LayoutResolver` accidentally hardcodes an IEEE-specific rule instead of reading it from data. **Root cause:** convenience shortcut under time pressure. **Debug:** grep the resolver for any literal standard name string — there should be none. **Recovery:** move the value into the JSON file immediately; this bug, left unfixed, is exactly what breaks Day 9's "new standard = new file" promise.
- **Bug:** Citation numbering doesn't stay stable if sections are reordered. **Root cause:** numbering computed per-section instead of once globally across the whole document during formatting. **Recovery:** compute the full reference list and numbering as a single pass over the entire `Draft`, not incrementally per section.
- **Bug:** Template JSON fails to load with a cryptic parser error. **Root cause:** hand-written JSON has a trailing comma or missing quote. **Recovery:** validate all template files through the same schema validator as config files (Day 3's utility), and add a "list all templates and validate on startup" check.

### 📸 Expected Screenshots
1. `ieee_us_letter.json` file contents (for documentation of the data-driven approach).
2. Debug view of a resolved `FormattedDocument` (structure + stylesheet) for the sample draft.
3. Console output of the full `TemplateEngine` unit test suite passing, with test count visible.

### ✅ End-of-Day Checklist
- [ ] `FormattedDocument`/`StyleSheet` models implemented.
- [ ] `ieee_us_letter.json` complete and schema-valid.
- [ ] `TemplateEngine`, `LayoutResolver`, `CitationFormatter` implemented with zero standard-specific conditional branches.
- [ ] Full IEEE formatting pipeline runs against the real Day 4 draft, end-to-end, with no model call involved.
- [ ] Unit test suite for the Formatting Layer passes and requires no running model.
- [ ] `ITemplateEngine.validate()` catches at least one deliberately-broken test case correctly.

### ➡️ Handoff Notes
The deterministic Formatting Layer is real, tested, and proven against IEEE end-to-end, with the citation-formatter strategy pattern already generalized (even though only the `numeric` strategy is fully exercised today) so that adding `author-date` for APA/MLA/Chicago on Day 9 is additive, not a rewrite. **Day 6 takes today's `FormattedDocument` output and renders it into a real `.docx` file** — it should not need to ask the Formatting Layer for any additional information; if it does, that's a signal today's `FormattedDocument` schema was incomplete and should be extended (documented, not patched silently).

---

## Day 6

### 🎯 Objective
Build the **Export Layer**: `DocxExporter`/`DocxRenderer`, producing a real, valid, openable `.docx` file from the IEEE `FormattedDocument`. By end of day, the full pipeline **Notes → Generation → IEEE Formatting → DOCX Export** works end-to-end for the first time — this is the first fully demoable slice of the product.

### 📖 Learning Outcomes
- Mapping an abstract structural document tree to concrete OOXML (DOCX) structures: sections, styles, headers/footers, page setup.
- Handling structured non-prose content (tables, captions, references) without ever routing them through the LLM.
- Validating exported file integrity programmatically, not just by eyeballing it in Word.

### 🛠 Features to Build
- `IExporter` interface + `DocxExporter` implementation.
- `DocxRenderer`: structure → OOXML mapping (margins, fonts, spacing, page size, headings, body paragraphs).
- `StyleMapper`: `StyleSheet` → DOCX style definitions.
- Reference-list rendering (from `FormattedDocument.references`).
- Header/footer rendering per the IEEE template's rules.

### 📝 Step-by-Step Implementation Plan
1. In `shared/interfaces/`, define `IExporter { export(document: FormattedDocument, path: string): ExportResult }`.
2. In `export/`, implement `DocxExporter` as the concrete `IExporter` for DOCX — its only job is orchestration (call `DocxRenderer`, write to disk, return a result object); it must not make any formatting decisions itself.
3. In `export/`, implement `StyleMapper.map(styleSheet): DocxStyleDefinitions` — one-to-one translation of margins/font/spacing/page size into the DOCX library's style API.
4. In `export/`, implement `DocxRenderer.render(document: FormattedDocument): DocxFileBuffer` — iterates `FormattedDocument.sections`, applies heading levels per the resolved layout, renders body paragraphs, and appends the references section using `FormattedDocument.references` (already fully formatted by Day 5 — the exporter only lays them out, it does not format citation text itself).
5. Implement header/footer rendering according to IEEE's specific rules (as encoded in `ieee_us_letter.json`) as a discrete rendering step separate from body content rendering, per the Architecture Document's explicit guidance.
6. Add basic structured-content passthrough: if the sample fixture includes any tabular data, confirm it survives untouched from `NotesDocument` through to a native DOCX table (if the sample fixture has no tables, add one today specifically to exercise this path — do not let it go untested until a real user hits it).
7. Wire a full "Generate → Format → Export" action into the UI, chaining Day 4 → Day 5 → today's export into one user-facing flow for the first time.
8. Manually open the resulting `.docx` in Word/LibreOffice and visually verify margins, font, heading styles, and reference list match IEEE US Letter expectations.

### 📂 Folder Changes
Populate: `export/`.

### 📄 Files
| File | Purpose |
|---|---|
| `shared/interfaces/IExporter.*` | Contract for all export formats (DOCX now, PDF potentially in V2) |
| `export/DocxExporter.*` | Orchestrates DOCX export |
| `export/DocxRenderer.*` | Structure-to-OOXML mapping |
| `export/StyleMapper.*` | StyleSheet-to-DOCX-style translation |
| `tests/integration/end_to_end_ieee_pipeline/full_pipeline.test.*` | First full Notes→DOCX integration test |

### 📦 Libraries / Packages
- A local, free, offline DOCX-generation library appropriate to the chosen language/runtime (must not require any network call or paid license — verify this explicitly before adopting it, per NFR-6/NFR-1/NFR-3).
- **Installation notes:** confirm the library can run fully offline (some libraries fetch template assets remotely on first use — reject any that do).

### 🔗 Integrations
- `DocxExporter` depends only on `FormattedDocument` (Day 5) — it has zero dependency on `IModelClient`, `NotesValidator`, or any AI-layer component, which should be explicitly verified today (e.g., by running the export test suite with the model backend intentionally not running, to prove the independence claimed in the Architecture Document).
- This is the first day all layers (Presentation → Application → AI → Formatting → Export) are chained together in one live user flow.

### 🧪 Testing Tasks
- **Unit:** `StyleMapper` against the IEEE `StyleSheet` — verify each field maps to the correct DOCX style property.
- **Integration:** Full pipeline test — sample notes → real generation (or a cached `Draft` fixture to keep this test fast) → IEEE formatting → DOCX export → open and verify the file is valid (not just "exists," actually parseable as a well-formed DOCX).
- **Manual:** Open the exported file in both Microsoft Word and LibreOffice Writer (if available) to catch any renderer-specific compatibility issues.
- **Edge cases:** A section with a very long heading (line-wrap behavior); a reference list with 10+ entries (pagination/overflow behavior); an empty section (should the exporter skip it or render an empty heading — decide explicitly and document the decision).
- **Expected outputs:** A `.docx` file that opens without repair prompts in Word, with visually correct 1-inch margins, correct font, correct heading hierarchy, and a numerically-ordered IEEE reference list.

### 🐞 Common Bugs
- **Bug:** Exported DOCX opens with a "needs repair" warning. **Root cause:** malformed OOXML from an incomplete style mapping or missing required XML parts. **Debug:** inspect the library's validation/linting output if available; check for missing required style definitions. **Recovery:** ensure every `StyleSheet` field has a corresponding, complete DOCX style entry — don't leave any partially mapped.
- **Bug:** Margins look correct in one word processor but wrong in another. **Root cause:** unit conversion error (e.g., inches vs. twips/EMU mismatch in the underlying library). **Recovery:** add an explicit unit-conversion utility with unit tests, rather than passing raw numbers through assuming units line up.
- **Bug:** References list renders but numbering doesn't match in-text citation numbers. **Root cause:** Day 5's `CitationFormatter` numbering and today's rendering order diverged (e.g., references re-sorted alphabetically during export by mistake). **Recovery:** confirm `DocxRenderer` renders `FormattedDocument.references` in the exact order/numbering already decided by Day 5 — it must not re-derive or re-sort.

### 📸 Expected Screenshots
1. Exported DOCX opened in Word/LibreOffice showing the title page and first content page.
2. Zoomed view of the heading hierarchy and body text styling.
3. Reference list page showing correct IEEE numeric formatting.
4. Full "Generate → Format → Export" UI flow, screenshotted at each stage.

### ✅ End-of-Day Checklist
- [ ] `IExporter`/`DocxExporter`/`DocxRenderer`/`StyleMapper` implemented.
- [ ] Full Notes→Generation→IEEE-Formatting→DOCX pipeline runs end-to-end without manual intervention.
- [ ] Exported file opens cleanly (no repair prompts) in at least one real word processor.
- [ ] Margins, fonts, headings, and references visually verified against IEEE US Letter requirements.
- [ ] Export layer's independence from the AI layer explicitly verified (export test passes with model backend stopped, using a cached `Draft` fixture).

### ➡️ Handoff Notes
**This is the first day the product's core acceptance criterion is met**: "given structured notes and a selected template standard, the system produces a complete draft with that standard's formatting correctly applied" — for IEEE, fully. This is a legitimate demo-ready milestone if the schedule were to compress from here. No session persistence exists yet (a fresh app restart loses all state), and Reviewer Mode doesn't exist yet. **Day 7 adds session save/load/autosave** on top of this now-working pipeline, without touching Generation, Formatting, or Export logic at all — it only needs to serialize/deserialize the data objects those layers already produce and consume.

---

## Day 7

### 🎯 Objective
Build the full **Session/Persistence Layer**: manual save, autosave, and reload-with-recovery, so that a user's notes, draft, template selection, and voice profile survive an app restart without data loss — directly satisfying FR-6/FR-7 and their acceptance criteria.

### 📖 Learning Outcomes
- Designing atomic, crash-safe local file writes (temp-file-then-replace pattern).
- Schema-versioning a serialization format from day one, even when only one version currently exists.
- Recovery-oriented error handling: what to do when a save file is corrupt, not just when it's missing.

### 🛠 Features to Build
- `ISessionStore` interface + `SessionStore` implementation (save/load).
- `AutosaveScheduler` (debounced, triggered on meaningful state changes).
- `session_schema_v1.json` schema definition + validation.
- Corruption recovery flow (fall back to last autosave temp file).
- UI wiring: manual "Save"/"Save As"/"Open Project" actions, and an autosave indicator.

### 📝 Step-by-Step Implementation Plan
1. In `persistence/schema/`, define `session_schema_v1.json` describing the full serialized shape: `{ schemaVersion, notesDocument, draft, templateSelection, voiceProfileRef, reviewerFeedback, metadata: { lastExport, lastModified } }`.
2. In `shared/interfaces/`, define `ISessionStore { save(state): void, load(path): SessionState, autosave(state): void }`.
3. In `persistence/`, implement `SessionStore`:
   - `save()`: serialize the full in-memory state to a temp file, then atomically rename/replace the real project file — never write directly over the existing good file.
   - `load()`: read the file, validate against `session_schema_v1.json`, and only then deserialize into real in-memory objects (`NotesDocument`, `Draft`, etc. — reusing the exact models from Days 2–5 unchanged).
   - On validation failure: attempt to load the last known autosave temp file before surfacing a corruption error to the user.
4. In `persistence/`, implement `AutosaveScheduler` — a debounce timer (e.g., a few seconds after the last state-changing action) that calls `SessionStore.autosave()`; wire it to fire after: note edits, draft generation completing, template selection changes, and reviewer runs.
5. Wire "Save," "Save As," and "Open Project" actions into the Presentation layer, plus a small non-intrusive autosave status indicator (e.g., "Saved" / "Saving…" / "Unsaved changes").
6. Deliberately corrupt a test save file (truncate it mid-write, or hand-edit invalid JSON into it) and verify the recovery flow: does it fall back to autosave correctly, and if that also fails, does it report corruption clearly rather than silently loading a broken/partial state?
7. Run a full save → close app → reopen app → reload cycle multiple times against the real pipeline output from Day 6, confirming zero data loss on every field.

### 📂 Folder Changes
Populate: `persistence/`, `persistence/schema/`.

### 📄 Files
| File | Purpose |
|---|---|
| `shared/interfaces/ISessionStore.*` | Persistence contract |
| `persistence/SessionStore.*` | Save/load/atomic-write implementation |
| `persistence/AutosaveScheduler.*` | Debounced autosave trigger logic |
| `persistence/schema/session_schema_v1.json` | Versioned serialization schema |
| `tests/unit/persistence/SessionStore.test.*` | Round-trip serialization tests |
| `tests/unit/persistence/CorruptionRecovery.test.*` | Deliberate-corruption recovery tests |

### 📦 Libraries / Packages
- Standard filesystem APIs of the chosen language/runtime — no new external dependency should be needed for atomic file writes (temp-file-then-rename is a standard OS-level pattern, not a library feature).
- Reuse the same JSON schema validator adopted on Day 3/5 for consistency — do not introduce a second validation library.

### 🔗 Integrations
- `SessionStore` depends on, but does not modify, every shared model from Days 2–6 (`NotesDocument`, `Draft`, `FormattedDocument` if cached, `TemplateSelection`, `VoiceProfile` reference, `ReviewerFeedback` — the last of these is a forward reference to Day 8's model; today, reserve a field for it in the schema even though `ReviewerEngine` doesn't exist until tomorrow).
- `AutosaveScheduler` hooks into the Application layer's workflows (`GenerationWorkflow`, template-selection change handlers) as an event listener, not by those workflows calling persistence directly — keeping persistence a cross-cutting concern rather than tangled into business logic.

### 🧪 Testing Tasks
- **Unit:** Full round-trip save→load equality test against the Day 6 real pipeline output (notes, draft, template selection) — every field must match exactly after reload.
- **Integration:** Autosave fires after a note edit, after draft generation, and after a template change; verify timing (not too aggressive, not so lazy that a crash loses meaningful work).
- **Manual:** Kill the application process mid-autosave (simulate a crash) and confirm the last good autosave still loads correctly on next launch.
- **Edge cases:** Loading a file with a newer `schemaVersion` than the app supports (should fail clearly, not attempt a silent best-effort parse); loading a completely empty file; loading a file with valid JSON but the wrong schema shape entirely.
- **Expected outputs:** Zero data loss across at least 5 repeated save/close/reopen cycles (this directly maps to the PRD's success metric: "Session save/load succeeds with no data loss across repeated test cycles").

### 🐞 Common Bugs
- **Bug:** Autosave fires too frequently, causing UI stutter or excessive disk I/O. **Root cause:** debounce window too short, or triggered on every keystroke instead of on meaningful state transitions. **Recovery:** widen the debounce window and confirm it's triggered on discrete events, not continuous ones.
- **Bug:** A crash during `save()`'s temp-file rename leaves both an old file and an orphaned temp file, and reload picks the wrong one. **Root cause:** rename isn't truly atomic on the target OS/filesystem, or cleanup logic runs before the rename completes. **Recovery:** verify the specific atomic-rename guarantee of the target OS; add a startup check that resolves any leftover temp files deterministically (prefer the most recently completed write).
- **Bug:** Deserialized `Draft`/`NotesDocument` objects don't behave identically to freshly-generated ones (e.g., missing a method or losing object identity assumptions elsewhere in the app). **Root cause:** deserialization producing plain data objects instead of properly reconstructed typed instances. **Recovery:** ensure `SessionStore.load()` reconstructs objects through the same constructors/factories used elsewhere, not raw JSON-parsed objects passed around directly.

### 📸 Expected Screenshots
1. UI showing the autosave status indicator during an active edit.
2. "Save As" dialog and resulting project file on disk.
3. Successful reload of a saved project, showing notes/draft/template selection all restored.
4. The corruption-recovery flow in action (intentionally broken file → recovery message shown).

### ✅ End-of-Day Checklist
- [ ] `ISessionStore`/`SessionStore` implemented with atomic writes.
- [ ] `AutosaveScheduler` wired to all relevant state-changing events.
- [ ] `session_schema_v1.json` defined and validated on load.
- [ ] Corruption recovery flow tested against a deliberately broken file.
- [ ] Zero data loss confirmed across at least 5 repeated save/reload cycles.
- [ ] Reserved schema field present for Day 8's `ReviewerFeedback`, even though unused today.

### ➡️ Handoff Notes
Session persistence is complete and proven robust against both normal use and deliberate corruption. The full pipeline (Notes → Generation → Formatting → Export) from Days 2–6 now survives app restarts with no data loss. **Day 8 adds Reviewer Mode** (`ReviewerEngine`) as a new, independent AI-layer component operating on the persisted `Draft`, and also uses Day 8 to harden error handling across the whole pipeline built so far (missing model, export failures, template mismatches) — none of which requires touching Session, Formatting, or Export code, only wrapping existing calls with the error-handling patterns defined in the Architecture Document §12.

---

## Day 8

### 🎯 Objective
Build **Reviewer Mode** (`ReviewerEngine`) end-to-end, and perform a dedicated **error-handling hardening pass** across the entire pipeline built Days 2–7, implementing every failure-mode pattern defined in the Architecture Document's Error Handling Strategy (§12).

### 📖 Learning Outcomes
- Designing a second, distinct AI pipeline (critique-framed, not production-framed) that reuses `IModelClient`/`OutputCleaner` but not `PromptBuilder`'s generation-specific templates.
- Systematic error-handling hardening as a discrete engineering activity, not an afterthought bolted on per-bug.
- Ensuring feedback is traceable to actual draft content rather than generic, to satisfy the PRD's explicit non-genericness acceptance criterion.

### 🛠 Features to Build
- `ReviewerEngine.review(draft): ReviewerFeedback[]`.
- Reviewer-specific prompt templates (critique framing, distinct from generation templates).
- `ReviewerFeedback` shared data model.
- Reviewer Panel UI (display feedback per section).
- Full error-handling pass: missing-model pre-flight check, export-failure recovery, corrupt-session fallback (already partly done Day 7 — extend here), template-mismatch reporting.

### 📝 Step-by-Step Implementation Plan
1. In `shared/models/`, define `ReviewerFeedback { sectionId, issue, suggestion, severity }`.
2. In `ai/reviewer/SectionPromptTemplates/` (or a `reviewer/` subfolder under prompts), author critique-framed prompt templates distinct from Day 4's generation templates — instruct the model to identify weaknesses (clarity, argument strength, unsupported claims, style inconsistency) per section, referencing actual section content directly in the prompt so feedback is inherently tied to that content.
3. In `ai/reviewer/`, implement `ReviewerEngine.review(draft: Draft): ReviewerFeedback[]` — iterates `Draft.sections`, builds a critique prompt per section (reusing `IModelClient`, not `PromptBuilder`'s generation-specific logic), cleans output via `OutputCleaner`, and parses/structures it into `ReviewerFeedback` entries.
4. Wire the reserved `ReviewerFeedback[]` field from Day 7's session schema to actually populate now, and confirm `AutosaveScheduler` fires after a reviewer run.
5. Build a minimal Reviewer Panel UI showing feedback grouped by section, with severity indicated.
6. **Error-handling hardening pass**, one item at a time, each with an explicit test:
   - Missing/unreachable model: add a pre-flight check in `GenerationWorkflow`/`ReviewerEngine` startup path that gives a clear, actionable message ("Model failed to respond — check that Ollama is running") rather than surfacing a raw exception.
   - Export failure: wrap `DocxExporter`'s calls so a failure reports which section/element failed, and confirm the `FormattedDocument` is preserved for retry without redoing generation.
   - Corrupt session: verify Day 7's fallback-to-autosave logic reports a clear message if *both* the main file and the autosave fail.
   - Template mismatch: confirm `TemplateEngine.validate()`'s specific error messages (Day 5) actually surface all the way up to the UI, not just logged internally.
7. Deliberately trigger each failure mode manually and confirm the user-facing message is specific and actionable, not a raw stack trace.

### 📂 Folder Changes
Populate: `ai/reviewer/`. Add `ReviewerFeedback` to `shared/models/`.

### 📄 Files
| File | Purpose |
|---|---|
| `shared/models/ReviewerFeedback.*` | Structured feedback data model |
| `ai/reviewer/ReviewerEngine.*` | Critique generation logic |
| `ai/reviewer/ReviewerPromptTemplates/*.json` | Critique-framed prompt templates |
| `app/views/ReviewerPanelView.*` | Displays feedback per section |
| `tests/integration/ReviewerEngine.test.*` | Mocked and real-model reviewer tests |
| `docs/adr/ADR-007-error-handling-pass.md` | Records each hardened failure mode and its test |

### 📦 Libraries / Packages
- No new libraries — Reviewer Mode reuses Day 3's `IModelClient` and Day 4's `OutputCleaner` entirely. This is a deliberate confirmation that the architecture's layering pays off: a whole new feature (FR-8) requires zero new infrastructure.

### 🔗 Integrations
- `ReviewerEngine` depends on `IModelClient` (Day 3) and `OutputCleaner` (Day 4), and consumes `Draft` (Day 4/7) — it does not depend on `TemplateEngine` or `DocxExporter` at all, correctly reflecting that review happens on content, before or independent of formatting.
- Error-handling hardening touches `GenerationWorkflow`, `DocxExporter`, `SessionStore`, and `TemplateEngine` only at their existing public interface boundaries — no internal logic from Days 2–7 should need restructuring, only additional guard clauses and clearer error propagation.

### 🧪 Testing Tasks
- **Unit:** `ReviewerFeedback` parsing from raw model text into structured entries, against several raw-output shapes (well-formed, malformed, empty).
- **Integration:** `ReviewerEngine.review()` against the real Day 6 draft, confirming feedback references specific section content (manually verify at least 2 pieces of feedback quote or clearly reference something specific from the actual text, not generic advice like "improve clarity").
- **Manual:** Trigger each of the four hardened failure modes deliberately (stop the model service, corrupt an export path, corrupt both session files, select a mismatched template) and confirm every one produces a specific, actionable message.
- **Edge cases:** Running Reviewer Mode on a very short/thin draft section; running it twice in a row (confirm feedback isn't duplicated or the second run doesn't corrupt the first run's stored feedback).
- **Expected outputs:** A `ReviewerFeedback[]` list with content-specific entries per section, and four confirmed, gracefully-handled failure scenarios with no raw stack traces reaching the UI.

### 🐞 Common Bugs
- **Bug:** Reviewer feedback is generic ("Consider improving clarity") rather than content-specific. **Root cause:** critique prompt didn't include enough actual section text, or was too abstract in its instructions. **Recovery:** ensure the prompt explicitly requires referencing a specific phrase/claim from the section; iterate the prompt template today, not later.
- **Bug:** Error messages are correctly specific in logs but a generic "Something went wrong" still reaches the UI. **Root cause:** a catch-all error boundary in the Presentation layer swallows the specific error object. **Recovery:** ensure error objects/messages propagate through, not just get replaced by a boundary's default message.
- **Bug:** Reviewer Mode accidentally triggers a full re-generation instead of operating on the existing draft. **Root cause:** workflow wiring mistake reusing `GenerationWorkflow` instead of the new `ReviewerEngine`. **Recovery:** verify at the UI action level that "Run Reviewer" is wired to `ReviewerEngine`, not `GenerationWorkflow`.

### 📸 Expected Screenshots
1. Reviewer Panel showing structured, section-specific feedback for the sample draft.
2. Each of the four hardened error scenarios, showing the specific user-facing message (4 screenshots).

### ✅ End-of-Day Checklist
- [ ] `ReviewerEngine` implemented, producing content-specific feedback.
- [ ] Reviewer Panel UI functional.
- [ ] `ReviewerFeedback` persists correctly through Day 7's session store.
- [ ] All four hardened error-handling scenarios tested and confirmed to produce specific, non-crashing messages.
- [ ] No raw stack traces or generic "something went wrong" messages reach the UI for any tested failure.

### ➡️ Handoff Notes
All PRD functional requirements (FR-1 through FR-10) now have a working implementation for at least the IEEE template standard, and the system fails gracefully across every identified failure mode. **Day 9 is dedicated to adding the remaining template standards/editions** (APA 6th/7th, MLA 8th/9th, Chicago 17th/18th) as stretch scope per the PRD's explicit depth-over-breadth allowance — if time runs short during Day 9, IEEE alone remains a fully complete, demoable V1, so nothing on Day 9 is a blocker for Day 10's integration/polish/demo-rehearsal work.

---

## Day 9

### 🎯 Objective
Extend template coverage beyond IEEE — **APA (6th, 7th), MLA (8th, 9th), Chicago (17th, 18th)** — by adding new `TemplateDefinition` data files and completing the `author-date` `CitationFormatter` strategy stubbed on Day 5. This day is explicitly scoped as stretch work: per the PRD, one complete standard (already done) is the success bar, and additional standards are additive value, not a requirement to hit Day 10.

### 📖 Learning Outcomes
- Validating that a data-driven template architecture actually delivers on its promise: adding six new templates should feel mechanical, not like new engineering.
- Handling a second citation strategy family (`author-date`) that both APA and MLA/Chicago share with variations.
- Prioritizing under time pressure: sequencing "most valuable first" when not all stretch items may finish.

### 🛠 Features to Build
- `apa_6.json`, `apa_7.json`, `mla_8.json`, `mla_9.json`, `chicago_17.json`, `chicago_18.json` template definitions.
- Complete the `author-date` `CitationFormatter` strategy (stubbed Day 5).
- A `footnote`/`note-bibliography` strategy if Chicago's specific citation approach requires it beyond simple author-date (verify against real Chicago rules before assuming author-date covers it).
- Template selector UI update to list all available standards/editions.

### 📝 Step-by-Step Implementation Plan
1. **Priority order for the day** (most-requested/most-distinct first, in case time runs out): APA 7th → APA 6th → MLA 9th → MLA 8th → Chicago 18th → Chicago 17th. This order front-loads the two most commonly required student/academic standards.
2. For each standard/edition: research (or confirm from existing knowledge) the specific mechanical rules — margins, font, spacing, heading style, title-page requirements, in-text citation format, reference-list title and entry format — and author the corresponding JSON file conforming to the exact schema used by `ieee_us_letter.json`.
3. Complete `CitationFormatter`'s `author-date` strategy: in-text `(Author, Year)` format, reference-list entry formatting per the `referenceEntryFormat` string in each template file.
4. If Chicago's specific rules require footnote/endnote-based citation rather than simple in-text author-date (verify this explicitly — do not assume), add a `footnote` strategy to the `CitationFormatter` registry — this is still "add a new strategy to an existing registry," not new architecture.
5. Run the full pipeline (generation → formatting → export) against the sample fixture for each newly added template, verifying a valid, correctly-styled DOCX comes out for each.
6. Update the Template Selector UI to list all completed standards/editions, defaulting to IEEE (the most-tested, most-reliable option) if none is explicitly chosen.
7. **Stop and reassess after each completed template**: if by mid-day only 2–3 of the six are done, that is an acceptable outcome per the PRD — do not sacrifice IEEE's stability or Day 10's integration time chasing all six. Log remaining templates as a clearly-scoped "not completed in V1" note rather than shipping a rushed, untested template file.

### 📂 Folder Changes
No new folders — only new files inside the existing `formatting/templates/` folder, and additions inside the existing `formatting/engine/CitationFormatter`.

### 📄 Files
| File | Purpose |
|---|---|
| `formatting/templates/apa_7.json`, `apa_6.json` | APA template definitions |
| `formatting/templates/mla_9.json`, `mla_8.json` | MLA template definitions |
| `formatting/templates/chicago_18.json`, `chicago_17.json` | Chicago template definitions |
| `formatting/engine/CitationFormatter.*` (updated) | Adds `author-date` (and `footnote` if needed) strategies |
| `tests/unit/formatting/templates_all.test.*` | Schema-validity + smoke test for every template file added today |

### 📦 Libraries / Packages
- None new — this day is purely data authoring plus completing an already-scaffolded strategy pattern, confirming the Day 5 architecture decision (ADR-002) was correct.

### 🔗 Integrations
- Every new template file plugs into the exact same `TemplateEngine`/`LayoutResolver`/`DocxExporter` pipeline built Days 5–6, unchanged. If any integration point *does* need to change to support a new standard, treat that as a signal the Day 5 schema had a gap — document and fix the schema (and retroactively confirm IEEE still works after the schema change), rather than special-casing the new standard.

### 🧪 Testing Tasks
- **Unit:** Schema validation for every new template file (reuse Day 3/5's validator).
- **Integration:** Full Notes→Generation→Format→Export run for each completed standard, using the same sample fixture, producing 6 (or however many were completed) distinct, valid `.docx` files.
- **Manual:** Open each exported file and visually spot-check against real APA/MLA/Chicago formatting references (title page presence/absence, running head rules, in-text citation style).
- **Edge cases:** A template with footnote-based citations (if implemented) — verify footnote numbering stays consistent across sections.
- **Expected outputs:** Every completed template produces a distinctly, correctly formatted DOCX from the identical underlying `Draft` — proving the formatting/content separation holds under variation, not just for the one standard it was built against.

### 🐞 Common Bugs
- **Bug:** A new template "mostly works" but breaks one shared assumption (e.g., `LayoutResolver` assumed all templates have a title page, but one doesn't). **Root cause:** Day 5's schema/resolver logic was implicitly built around IEEE's specific shape. **Recovery:** make the assumption explicit and conditional on the data (`titlePage.required: false`), not hardcoded as always-true.
- **Bug:** APA 6th and 7th are accidentally identical because differences weren't researched carefully. **Root cause:** copy-pasting one file to create the other without diffing the actual rule differences. **Recovery:** explicitly diff the two editions' real rules before finalizing both files.
- **Bug:** Running out of time mid-standard leaves a half-correct template file that silently produces a subtly wrong document. **Root cause:** rushing without the "stop and reassess" discipline from step 7 above. **Recovery:** better to ship fewer, fully-correct templates than more, partially-wrong ones — remove an incomplete template from the selector UI rather than leaving it selectable and broken.

### 📸 Expected Screenshots
1. Template Selector UI showing all completed standards/editions.
2. Side-by-side comparison of the same sample draft exported in IEEE vs. APA (or whichever second standard was completed) formatting.
3. Console/test output showing all new template files passing schema validation.

### ✅ End-of-Day Checklist
- [ ] At least one additional standard beyond IEEE fully completed and tested end-to-end (stretch target: all six, acceptable minimum: IEEE + 1–2 more per PRD's depth-over-breadth allowance).
- [ ] `author-date` citation strategy implemented and tested.
- [ ] Every completed template passes schema validation and produces a valid, openable DOCX.
- [ ] Any incomplete/untested templates explicitly removed from the UI selector rather than left in a broken state.
- [ ] IEEE (Day 5/6's original, fully-proven standard) reconfirmed still working after any shared-code changes made today.

### ➡️ Handoff Notes
Template coverage has been extended as far as time allowed, with IEEE remaining the fully-proven baseline regardless of how many additional standards were completed. **Day 10 is the final integration, polish, testing, and demo-rehearsal day** — it assumes the feature set is now frozen (no new templates, no new features) and focuses entirely on making the existing, working pipeline reliable, presentable, and crash-proof for a live, cold-start demo (NFR-7).

---

## Day 10

### 🎯 Objective
Final **integration, polish, full regression testing, and live demo rehearsal**. No new features. The goal is a stable, demoable, submittable Version 1.0 that satisfies every acceptance criterion in the PRD.

### 📖 Learning Outcomes
- Running a full acceptance-criteria checklist against a real system, treating the PRD's §10 as a literal test plan rather than a general guideline.
- Rehearsing a live, cold-start demo and hardening against exactly the failure modes a demo audience would notice.
- Making judgment calls about what "done" means for a 10-day solo build without slipping into further feature work.

### 🛠 Features to Build
None. This is explicitly a stabilization, testing, and polish day — building any new feature today is scope creep and directly contradicted by the PRD's own risk analysis ("solo 10-day builds fail most often on integration and polish time").

### 📝 Step-by-Step Implementation Plan
1. Run the full regression suite (all unit + integration tests from Days 2–9) and fix any regressions surfaced by Day 9's template additions before doing anything else.
2. Walk the PRD's Acceptance Criteria (§10) line by line as a literal checklist against the real running app — not a code review, an actual click-through:
   - Notes + IEEE (and any other completed standard) → correctly formatted complete draft.
   - Voice profile produces text recognizably distinct from generic AI output (manual judgment call, documented).
   - Save → reload restores everything with no data loss.
   - Reviewer Mode returns content-specific feedback.
   - DOCX export is valid and opens cleanly.
   - Entire workflow completes with no network connection active (literally disconnect the network and run the full flow once, to prove this rather than assume it).
   - Full demo scenario runs on the actual 4GB GPU target hardware without crashing.
3. UI polish pass: consistent spacing/labeling across Notes Editor, Draft View, Reviewer Panel, Template Selector, Save/Load — no functional changes, cosmetic only, time-boxed strictly (do not let this expand).
4. Prepare and rehearse the **exact demo script** end-to-end at least 3 times, cold (closing and reopening the app each time, not relying on warm in-memory state), timing each run.
5. Deliberately induce each of Day 8's four hardened failure modes one more time during rehearsal to confirm they still behave correctly after Day 9's changes.
6. Freeze the codebase (tag/branch a release candidate) once the full acceptance checklist passes three consecutive cold-start rehearsals with no crash.
7. Write the README, installation guide, and demo instructions (see Deployment Checklist below) — the last engineering task of the 10 days.

### 📂 Folder Changes
No structural changes. Add/finalize: `README.md`, `docs/installation_guide.md`, `docs/demo_script.md` at the repository root/`docs/`.

### 📄 Files
| File | Purpose |
|---|---|
| `README.md` | Primary project documentation for GitHub |
| `docs/installation_guide.md` | Step-by-step setup for a new machine |
| `docs/demo_script.md` | The exact rehearsed demo flow |
| `docs/acceptance_checklist_results.md` | Filled-in PRD §10 checklist with pass/fail evidence |
| `CHANGELOG.md` | Summary of what V1.0 includes (and explicitly what it excludes, per the Product Vision's §7 Exclusions) |

### 📦 Libraries / Packages
None new. Today is explicitly a zero-new-dependency day — any new library introduced this late is an unacceptable stability risk this close to a live demo.

### 🔗 Integrations
No new integrations. Today verifies that all existing integrations (Presentation ↔ Application ↔ AI ↔ Formatting ↔ Persistence ↔ Export ↔ Configuration) hold together under real, repeated, cold-start use — not just in isolated unit tests.

### 🧪 Testing Tasks
- **Regression:** Full test suite from all prior days, run fresh, zero failures tolerated.
- **Acceptance:** Every PRD §10 acceptance criterion checked off with actual evidence (screenshot, test log, or timed observation), recorded in `docs/acceptance_checklist_results.md`.
- **Manual (cold-start demo rehearsal):** Minimum 3 full cold runs of the entire notes→export flow, network disconnected, on the actual target 4GB GPU hardware, no crashes.
- **Edge cases (final sweep):** Extremely long notes document (stress-test generation time against NFR-5's "minutes, not tens of minutes" target); rapid repeated save actions; switching templates mid-session after a draft already exists.
- **Expected outputs:** Three consecutive clean cold-start demo runs; a fully checked-off acceptance criteria document; zero known open crashes.

### 🐞 Common Bugs
- **Bug:** Something that worked in isolated testing fails only during the full cold-start sequence. **Root cause:** a startup-ordering assumption (e.g., config loaded before the model daemon is confirmed running) that unit tests didn't exercise. **Recovery:** add an explicit startup sequence check; this is exactly the "integration and polish" risk both source documents warned about — treat it as expected, not alarming, and fix it methodically rather than panicking into new architecture.
- **Bug:** Demo works fine at a desk but stutters/lags in front of an audience due to GPU thermal throttling or background processes. **Root cause:** environmental, not code. **Recovery:** rehearse in conditions as close to the real demo environment as possible; close unnecessary background applications before demoing.
- **Bug:** A "polish" UI tweak accidentally breaks a working data-binding. **Root cause:** cosmetic changes touching functional code without adequate re-testing. **Recovery:** re-run the relevant test suite after every polish change, no matter how small it seems.

### 📸 Expected Screenshots
1. Full README rendered on GitHub (mock preview or actual repo view).
2. The completed acceptance-criteria checklist document.
3. Final polished UI — all four main views (Notes Editor, Draft/Reviewer, Template Selector, Export confirmation).
4. A successful cold-start demo run, screenshotted at each major step, network-disconnected indicator visible if the platform shows one.

### ✅ End-of-Day Checklist
- [ ] Full regression suite green.
- [ ] Every PRD §10 acceptance criterion checked off with evidence.
- [ ] Three consecutive clean cold-start demo rehearsals completed on target hardware.
- [ ] README, installation guide, and demo script written.
- [ ] Release candidate tagged/frozen.
- [ ] No new features or dependencies introduced today.

### ➡️ Handoff Notes
Version 1.0 is complete, tested against every PRD acceptance criterion, and demo-rehearsed cold on the actual target hardware. There is nothing further to build under this blueprint. Any future work belongs to Version 2.0 planning, using the extension points already architected in (see "Version 2 Preparation" below) — none of which require reopening or refactoring any V1 code.

---

# Deployment Checklist

- [ ] **Repository cleanup:** remove debug harnesses/scratch scripts not needed for the shipped app (or clearly mark them as dev tools in a `tools/` folder); remove any leftover placeholder `.gitkeep` files that now have real content; confirm `.gitignore` excludes local model files, session project files, and any local build artifacts.
- [ ] **README:** project description, tagline, screenshot, quick-start instructions, link to installation guide, explicit list of V1 scope and V1 exclusions (copied faithfully from the Product Vision §6/§7 so expectations are accurate).
- [ ] **Installation guide:** OS prerequisites, GPU/driver requirements (4GB minimum), Ollama or llama.cpp setup instructions, Qwen2.5-3B model download/setup steps, application install/run steps.
- [ ] **Requirements:** exact dependency list with versions (language runtime, DOCX library, HTTP client, test framework), hardware minimums restated plainly.
- [ ] **Environment setup:** step-by-step from a completely clean machine to a running app, validated by literally following it on a fresh environment if possible.
- [ ] **Sample project:** ship the Day 2 sample fixture as an example `.scholarforge` project new users can open immediately to see the product working without writing their own notes first.
- [ ] **Demo workflow:** the exact rehearsed Day 10 demo script, written clearly enough that someone other than the developer could follow it.
- [ ] **Testing checklist:** confirmation that the full automated test suite (unit + integration across all 9 days) is documented and runnable via one command.
- [ ] **Documentation checklist:** architecture document, PRD, product vision, and this blueprint all included in `docs/` alongside the README, so the repository is self-explanatory to a new reader.
- [ ] **Performance checklist:** documented, empirically-measured generation latency per section and total pipeline time on the target 4GB GPU (from Day 3's ADR and Day 10's stress testing).
- [ ] **Privacy checklist:** explicit statement (and architectural confirmation, per the Architecture Document §13) that no network calls occur outside localhost during normal operation; confirmed by the Day 10 network-disconnected test run.
- [ ] **Release checklist:** version tag applied, CHANGELOG finalized, release candidate branch frozen, final regression suite green.

---

# GitHub Structure

```
scholarforge/
├── app/                # Presentation layer
├── application/         # Use-case orchestration + validation
├── ai/                  # Prompting, model clients, voice profile, reviewer
├── formatting/          # Deterministic template engine + template data files
├── persistence/         # Session save/load/autosave
├── export/               # DOCX rendering
├── config/               # All configuration files + ConfigurationService
├── shared/               # Cross-layer data models and interfaces
├── tests/                # Unit, integration, and fixture data
├── tools/                # Dev-only scripts (e.g., Day 3's debug harness) — clearly separated from shipped app code
├── docs/
│   ├── architecture.md            # The approved Software Architecture Document
│   ├── prd.md                     # The approved PRD
│   ├── product_vision.md          # The approved Product Vision
│   ├── implementation_blueprint.md # This document
│   ├── installation_guide.md
│   ├── demo_script.md
│   ├── acceptance_checklist_results.md
│   └── adr/                       # One file per Architecture Decision Record
├── README.md
├── CHANGELOG.md
└── .gitignore
```

**Documentation organization rationale:** every planning artifact that governed this build (Vision, PRD, Architecture, this Blueprint) ships *inside* the repository under `docs/`, not just referenced externally — so the repository itself is a complete, self-contained record of why the system is shaped the way it is, which matters both for a solo developer returning to this project later and for anyone evaluating the project's engineering discipline (e.g., a reviewer, collaborator, or hiring manager).

---

# Milestone Tracking Table

| Day | Objective | Deliverables | Dependencies | Status Criteria |
|---|---|---|---|---|
| 2 | Project skeleton + Notes Module | Folder structure, `NotesDocument` model, Notes Editor UI, `NotesValidator` | None (Day 1 environment setup only) | Sample fixture validates cleanly; unit tests green |
| 3 | Configuration + AI connectivity | `ConfigurationService`, `IModelClient`, `OllamaClient`/`LlamaCppClient`, empirical hardware tuning | Day 2 (independent otherwise) | Real model responds within 4GB budget via debug harness |
| 4 | Prose generation pipeline | `PromptBuilder`, `VoiceProfile`, `OutputCleaner`, `GenerationWorkflow`, working `Draft` | Days 2–3 | Full `Draft` generated from sample notes, voice-profile influence visible |
| 5 | Deterministic Template Engine + IEEE | `TemplateEngine`, `LayoutResolver`, `CitationFormatter`, `ieee_us_letter.json` | Day 4 (`Draft`) | `FormattedDocument` for IEEE resolves correctly, zero model dependency in tests |
| 6 | DOCX Export | `DocxExporter`, `DocxRenderer`, `StyleMapper` | Day 5 (`FormattedDocument`) | Valid, correctly-styled `.docx` opens cleanly in Word/LibreOffice |
| 7 | Session persistence | `SessionStore`, `AutosaveScheduler`, recovery flow | Days 2–6 (serializes their outputs) | Zero data loss across 5+ save/reload cycles; corruption recovery verified |
| 8 | Reviewer Mode + error hardening | `ReviewerEngine`, four hardened failure-mode handlers | Days 3–4 (`IModelClient`, `OutputCleaner`), Day 7 (persistence of feedback) | Content-specific feedback produced; all four failure modes gracefully handled |
| 9 | Additional templates (stretch) | APA/MLA/Chicago template files, `author-date` strategy | Day 5 (`TemplateEngine`/`CitationFormatter`) | At least one additional standard fully working; IEEE reconfirmed unaffected |
| 10 | Integration, testing, demo rehearsal | Full regression pass, acceptance checklist, README/docs, 3 clean cold-start demos | All prior days | Three consecutive crash-free cold-start demo runs; every PRD §10 criterion checked with evidence |

---

# Risk Tracking Matrix

| Day | Risk | Probability | Impact | Mitigation | Fallback Plan |
|---|---|---|---|---|---|
| 2 | Notes schema is under-designed and needs late changes | Medium | Medium | Design `NotesDocument`/`SourceRef` conservatively, informed by what citation formatting will need (checked against Architecture Document §8 schema) | If a field is missing later, add it as an optional field with a migration-safe default rather than a breaking schema change |
| 3 | 4GB VRAM cannot reliably run Qwen2.5-3B at usable context length | Medium-High | High | Empirical testing on Day 3 itself, before any other AI-layer code is written | Reduce context window and/or increase quantization aggressiveness; worst case, generate shorter per-call sections (smaller note chunks) |
| 4 | Model output quality/voice fidelity is weak | Medium | Medium | Iterative prompt refinement against real output same-day; strong exemplar snippets in `VoiceProfile` | Accept voice preservation as approximate (per explicit PRD risk acknowledgment); do not over-invest fixing an inherently fuzzy target at the expense of later days |
| 5 | Data-driven template schema doesn't generalize cleanly to non-IEEE standards later | Medium | High (would undermine Day 9) | Design schema by cross-checking against at least a mental model of APA/MLA requirements even while only implementing IEEE today | If gaps appear Day 9, extend the schema (backward-compatibly) rather than special-casing new standards in code |
| 6 | DOCX library produces invalid/uncompliant files | Low-Medium | High | Validate output opens cleanly in a real word processor same-day, not just "library didn't throw" | Switch DOCX library early if fundamental issues appear — Day 6 is early enough to absorb this switch; Day 9+ would not be |
| 7 | Autosave/atomic-write logic has a subtle crash-safety bug | Medium | High (directly threatens "no data loss" acceptance criterion) | Explicit deliberate-corruption and simulated-crash testing same day | Widen autosave interval and add a manual "Save" reminder prompt as a safety net if atomic-write can't be fully hardened in time |
| 8 | Reviewer feedback stays generic despite prompt tuning | Medium | Medium | Require prompts to reference specific section content explicitly; manually review real output before calling it done | Document as a known limitation rather than blocking the day — Reviewer Mode existing and running is the FR-8 requirement; feedback quality has a manual-review acceptance bar, not an automated one |
| 9 | Time runs out before all six template standards are complete | High | Low (explicitly acceptable per PRD) | Strict priority ordering (APA 7th first) and a "stop and reassess" checkpoint after each template | Ship IEEE alone as V1's complete standard, exactly as the PRD explicitly permits — this is a non-emergency, expected outcome |
| 10 | A late-discovered bug threatens demo stability | Medium | High | Full regression suite + 3 rehearsed cold-start runs specifically designed to surface this before the real demo | If a bug can't be fixed safely in the time remaining, disable/hide the specific feature or template edition that triggers it rather than risk a live crash — a smaller, stable demo beats a broader, fragile one |

---

# Quality Gates

Each day's work must clear its gate before the next day begins. If a gate is not met, the next day's plan should not proceed as written — the developer should first close the gap (using that day's own "Common Bugs"/"Handoff Notes" guidance) before starting new work.

| Gate (end of Day) | Must be true before proceeding |
|---|---|
| 2 | Sample notes fixture validates with zero issues; validator unit tests all pass |
| 3 | Real local model returns a real response within the 4GB VRAM budget via the debug harness |
| 4 | A complete, readable `Draft` is generated end-to-end from the sample fixture with the real model |
| 5 | `FormattedDocument` for IEEE resolves correctly and all Formatting Layer tests pass without the model running |
| 6 | Exported `.docx` opens without a repair prompt in a real word processor, matching IEEE layout expectations |
| 7 | Zero data loss confirmed across 5+ save/reload cycles, including one deliberate-corruption recovery test |
| 8 | `ReviewerEngine` produces content-specific feedback; all four hardened error scenarios behave correctly |
| 9 | At least one additional standard beyond IEEE is fully working end-to-end, and IEEE itself still passes regression |
| 10 | Three consecutive crash-free cold-start demo runs on target hardware, and every PRD §10 acceptance criterion is checked with evidence |

---

# Version 2 Preparation

The following hooks already exist in the Version 1 architecture (per the Architecture Document §14) and require **no V1 rework** to extend later — this list is a confirmation of what's already there, not new design work:

- **`IModelClient` abstraction** — a LoRA-adapted or fine-tuned model becomes a new implementation or a config change, not a rewrite of `PromptBuilder`/`GenerationWorkflow`.
- **Data-driven `TemplateDefinition` schema** — new journal templates or citation standards are new JSON files, following the exact pattern established across IEEE/APA/MLA/Chicago in Days 5 and 9.
- **`IExporter` interface** — a future `PdfExporter` sits alongside `DocxExporter` behind the same contract; `DocxRenderer` is untouched.
- **`ISessionStore` interface** — a future `CloudSessionStore` sits alongside the local `SessionStore` behind the same contract, selected by config.
- **`INotesSource`-shaped extension point** (implied by `NotesModule`'s clean boundary) — a future citation-manager import (Zotero/Mendeley) or semantic search feature attaches upstream of the Notes Module without touching validation, generation, formatting, or export.
- **Config-driven, registry-based backend/template selection throughout** — the pattern already used for model backend and template selection is the same pattern a future plugin architecture would formalize; V1 already proves the registry approach works end-to-end under real use.

No V2 feature listed in the Product Vision requires reopening this blueprint's Day 2–10 work — that is the direct payoff of the architectural boundaries established from Day 2 onward.
