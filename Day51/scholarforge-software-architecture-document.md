# ScholarForge — Software Architecture Document (V1.0)

**Source of truth:** ScholarForge Version 1.0 Product Vision (Approved) + ScholarForge PRD (Draft for Build)
**Author role:** Principal Software Architect
**Status:** Architecture blueprint for a 10-day solo build
**Scope discipline:** This document adds no features, changes no scope, and reinterprets no business decision. It exists solely to define *how* the approved requirements get built.

---

## 1. Executive Architecture Summary

ScholarForge is architected as a **layered, offline-first monolith with hard internal module boundaries** — not a distributed system, not a set of separately deployed services. A single-process desktop/local application is the correct shape for this product: it must run on a 4GB GPU with no network dependency, it has exactly one user at a time (per PRD §13 assumption of single-session, single-user), and it must be buildable solo in 10 days. Anything resembling microservices, message queues, or multi-process orchestration would burn schedule on infrastructure the product doesn't need.

The one architectural decision that matters more than any other, and that every other decision in this document serves, is:

> **The AI Generation Layer and the Deterministic Formatting Layer never share responsibility for the same output.** The LLM only ever produces *language*. Every formatting decision — margins, headings, citation punctuation, page size, DOCX structure — is computed by deterministic code that has no dependency on model output quality, phrasing, or behavior.

This separation is chosen because it is the only way to satisfy NFR-4 ("formatting rules deterministic, not subject to LLM output variance") while still getting the benefit of a language model for prose generation. It also happens to be good software engineering independent of the AI angle: it means the formatting engine is unit-testable without ever invoking the model, and the model layer can be swapped, upgraded, or replaced (Ollama ↔ llama.cpp, Qwen2.5-3B ↔ a future fine-tuned model) without touching a single line of template logic.

The system is organized into seven layers (Presentation, Application, AI, Formatting, Persistence, Export, Configuration), communicating through narrow interfaces and plain data objects (a `Notes` object, a `Draft` object, a `RenderedDocument` object) rather than shared mutable state. This gives a solo developer a mental model simple enough to hold in their head on day 10 as well as day 1, while leaving clean extension points for every item already flagged in the V2.0 Vision (LoRA, RAG, citation manager integration, PDF export, plugins, cloud sync) — none of which require touching V1 code, only adding new implementations behind existing interfaces.

---

## 2. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      PRESENTATION LAYER                         │
│   Notes Editor · Template Picker · Draft View · Reviewer Panel  │
└───────────────────────────────┬───────────────────────────────┘
                                 │  (UI events, view models)
┌───────────────────────────────▼───────────────────────────────┐
│                      APPLICATION LAYER                          │
│   Use-case orchestrators (Workflows) · Validation · DI wiring   │
└───────┬───────────────┬───────────────┬───────────────┬────────┘
        │               │               │               │
┌───────▼──────┐ ┌───────▼──────┐ ┌──────▼───────┐ ┌─────▼──────┐
│   AI LAYER    │ │ FORMATTING   │ │ PERSISTENCE  │ │  EXPORT    │
│ Prompt Build  │ │   LAYER      │ │   LAYER      │ │  LAYER     │
│ Model Client  │ │ Template     │ │ Session      │ │ DOCX       │
│ Reviewer Gen  │ │ Engine       │ │ Store        │ │ Builder    │
│ Output Clean  │ │ Style Rules  │ │ Autosave     │ │            │
└───────┬───────┘ └──────┬───────┘ └──────┬───────┘ └─────┬──────┘
        │                │                │                │
        └────────────────┴────────┬───────┴────────────────┘
                                   │
                      ┌────────────▼────────────┐
                      │   CONFIGURATION LAYER    │
                      │ Model cfg · Voice cfg     │
                      │ Template registry · Paths │
                      └───────────────────────────┘
```

### Layer Responsibilities

| Layer | Owns | Never Does |
|---|---|---|
| **Presentation** | Rendering UI, capturing input, displaying draft/feedback | Business logic, formatting math, direct model calls, direct file I/O |
| **Application** | Orchestrating use cases (generate draft, run reviewer, save session, export), validating inputs, wiring dependencies | Knowing *how* the AI or formatting layers work internally |
| **AI Layer** | Prompting, model invocation, response cleaning, voice-profile injection, Reviewer Mode generation | Deciding margins, headings, citation punctuation, file formats |
| **Formatting Layer** | Template rules per standard/edition, deterministic layout decisions, citation structure | Generating prose, calling the model, touching the filesystem directly |
| **Persistence Layer** | Session serialization, autosave, load/recovery | Formatting decisions, AI calls |
| **Export Layer** | Assembling the final DOCX from formatted+generated content | Deciding *what* the content or formatting should be — it only renders what it's given |
| **Configuration Layer** | All tunables: model settings, voice profile, template registry, storage paths | Any runtime business logic |

This is a classic **Ports & Adapters (Hexagonal) shape** applied pragmatically: the Application Layer defines interfaces (ports); AI, Formatting, Persistence, and Export are adapters behind those interfaces. This is exactly what makes the "swap Ollama for llama.cpp" or "add PDF export in V2" changes additive rather than invasive.

---

## 3. Complete Module Breakdown

### 3.1 `NotesModule` (Presentation + Application)

- **Purpose:** Capture structured research notes (FR-1).
- **Responsibilities:** Provide structured input fields (e.g., sections, claims, sources, quotes), validate minimum structure before generation is allowed, hand a normalized `NotesDocument` to the Application Layer.
- **Inputs:** User keystrokes/form input.
- **Outputs:** `NotesDocument` (plain data object: ordered list of `NoteSection { heading, bullets[], sourceRefs[] }`).
- **Dependencies:** None outside Presentation/Application boundary.
- **Public Interface:** `INotesRepository.getCurrent(): NotesDocument`, `INotesValidator.validate(doc): ValidationResult`.
- **Internal Components:** Section editor widget, source-tagging widget, structure validator.
- **Future Extension (V2):** Import from reference managers (Zotero/Mendeley) — plugs in as an alternate `INotesSource` adapter, no change to the editor or validator contracts.

### 3.2 `GenerationWorkflow` (Application Layer)

- **Purpose:** Orchestrate the notes → draft use case (FR-2).
- **Responsibilities:** Validate notes are present, invoke `PromptBuilder`, call `IModelClient.generate()`, pass raw model output through `OutputCleaner`, assemble a `DraftSection`, hand it to the Formatting Layer for placement into the template structure.
- **Inputs:** `NotesDocument`, `VoiceProfile`, `TemplateSelection`.
- **Outputs:** `Draft` (ordered `DraftSection[]`, each with cleaned prose + a pointer to its source `NoteSection`).
- **Dependencies:** `AI Layer` (via `IModelClient`, `IPromptBuilder`), `Configuration Layer` (voice profile, model settings).
- **Public Interface:** `generateDraft(notes, config): Draft`.
- **Internal Components:** Retry/backoff wrapper around model calls, section-by-section generation loop (keeps context small enough for 4GB VRAM per NFR-2/NFR-5).
- **Future Extension:** Swap in a RAG-augmented prompt builder without changing the workflow's public contract.

### 3.3 `AI Layer` — `PromptBuilder`, `ModelClient`, `OutputCleaner`, `ReviewerEngine`

- **Purpose:** All language generation (FR-2, FR-5, FR-8).
- **Responsibilities:**
  - `PromptBuilder`: composes system + task prompts from note content, the `VoiceProfile`, and section-type templates (e.g., "Introduction," "Methodology"). Enforces prompt-length budgets appropriate to a 3B model on 4GB VRAM.
  - `ModelClient`: a thin adapter interface (`IModelClient`) with two concrete implementations — `OllamaClient` and `LlamaCppClient` — chosen at config time. Neither the workflow nor the prompt builder knows or cares which backend is active.
  - `OutputCleaner`: strips model artifacts (stray markdown, repeated boilerplate, incomplete sentences at truncation boundaries) before content ever reaches the Formatting Layer. This is the seam that guarantees FR-4/NFR-4 — formatting code downstream never has to "understand" model quirks.
  - `ReviewerEngine`: a distinct prompt/response pipeline for FR-8, operating on a completed `Draft` rather than raw notes, producing structured `ReviewerFeedback { section, issue, suggestion }[]`.
- **Inputs:** `NotesDocument` or `Draft`, `VoiceProfile`, `ModelConfig`.
- **Outputs:** Cleaned prose strings, `ReviewerFeedback[]`.
- **Dependencies:** Local model runtime only (Ollama or llama.cpp process/library). No network calls (NFR-1, NFR-3).
- **Public Interfaces:** `IModelClient.generate(prompt, options): string`, `IPromptBuilder.build(section, context): Prompt`, `IReviewerEngine.review(draft): ReviewerFeedback[]`.
- **Future Extension:** A fine-tuned or LoRA-adapted model becomes a third `IModelClient` implementation; RAG becomes an additional step inside `PromptBuilder` behind the same interface.

### 3.4 `TemplateEngine` (Formatting Layer)

- **Purpose:** Own all deterministic formatting rules for every supported standard/edition (FR-3, FR-4, FR-10).
- **Responsibilities:** Given a `Draft` and a `TemplateSelection` (standard + edition + page size where applicable), produce a `FormattedDocument` — a structural tree annotated with layout instructions (margins, heading levels, citation markers, spacing rules) but containing zero LLM-authored formatting decisions.
- **Inputs:** `Draft`, `TemplateSelection`.
- **Outputs:** `FormattedDocument` (structural tree + a `StyleSheet` resolved from data-driven template rule files).
- **Dependencies:** `Configuration Layer` (template registry — see §8).
- **Public Interfaces:** `ITemplateEngine.apply(draft, selection): FormattedDocument`, `ITemplateRule.validate(document): ValidationResult`.
- **Internal Components:** One `TemplateDefinition` per standard/edition (data, not code — see §8), a shared `CitationFormatter` strategy, a shared `LayoutResolver`.
- **Future Extension:** New standards or editions are added as new data files, not new code paths.

### 3.5 `SessionManager` (Persistence Layer)

- **Purpose:** Save/load session state without data loss (FR-6, FR-7).
- **Responsibilities:** Serialize `{ NotesDocument, Draft, TemplateSelection, VoiceProfile, ReviewerFeedback[], metadata }` to a single local project file; deserialize back into equivalent in-memory objects; autosave on a timer/change trigger; validate integrity on load and offer recovery on corruption.
- **Inputs:** In-memory application state.
- **Outputs:** A `.scholarforge` project file (structured, e.g., JSON or a zipped JSON+assets bundle).
- **Dependencies:** Local filesystem only.
- **Public Interfaces:** `ISessionStore.save(state): void`, `ISessionStore.load(path): SessionState`, `ISessionStore.autosave(state): void`.
- **Future Extension:** Draft versioning/history (V2) is additive — new fields in the same file format, or a companion version-history file, without breaking existing load logic (schema-versioned files, see §9).

### 3.6 `ExportEngine` (Export Layer)

- **Purpose:** Produce the final DOCX (FR-9).
- **Responsibilities:** Take a `FormattedDocument` and render it into a valid `.docx` file — headers, footers, margins, fonts, spacing, page size, references section, captions, tables/images if present in notes.
- **Inputs:** `FormattedDocument`.
- **Outputs:** `.docx` file on local disk.
- **Dependencies:** A DOCX-generation library (local, zero-cost, no network).
- **Public Interfaces:** `IExporter.export(document, path): ExportResult`.
- **Internal Components:** `DocxRenderer` (structure→OOXML mapping), `StyleMapper` (StyleSheet → DOCX style definitions).
- **Future Extension:** `IExporter` gets a second implementation (`PdfExporter`) in V2 without touching `DocxRenderer` or anything upstream.

### 3.7 `ConfigurationService` (Configuration Layer)

- **Purpose:** Single source for all tunables (model settings, voice profile, template registry, reviewer options, export settings, project paths).
- **Responsibilities:** Load/validate config at startup, expose typed accessors, prevent any other module from hardcoding a setting.
- **Inputs:** A local config file (and template rule files it points to).
- **Outputs:** Strongly-typed config objects consumed by every other layer.
- **Dependencies:** Local filesystem.
- **Public Interfaces:** `IConfig.get<T>(key): T`.
- **Future Extension:** New config sections (RAG settings, cloud-sync credentials) just extend the schema.

---

## 4. Folder Structure

```
scholarforge/
├── app/                          # Presentation layer (UI)
│   ├── views/
│   │   ├── NotesEditorView
│   │   ├── TemplatePickerView
│   │   ├── DraftView
│   │   └── ReviewerPanelView
│   └── viewmodels/               # Thin view-state adapters, no business logic
│
├── application/                  # Application layer: use-case orchestration
│   ├── workflows/
│   │   ├── GenerationWorkflow
│   │   ├── ReviewWorkflow
│   │   ├── ExportWorkflow
│   │   └── SessionWorkflow
│   └── validation/
│       └── NotesValidator
│
├── ai/                           # AI layer
│   ├── prompt/
│   │   ├── PromptBuilder
│   │   └── SectionPromptTemplates/     # data files, one per section type
│   ├── client/
│   │   ├── IModelClient
│   │   ├── OllamaClient
│   │   └── LlamaCppClient
│   ├── cleaning/
│   │   └── OutputCleaner
│   ├── voice/
│   │   └── VoiceProfile
│   └── reviewer/
│       └── ReviewerEngine
│
├── formatting/                   # Deterministic formatting layer
│   ├── engine/
│   │   ├── ITemplateEngine
│   │   ├── TemplateEngine
│   │   ├── CitationFormatter
│   │   └── LayoutResolver
│   └── templates/                # DATA, not code
│       ├── ieee_us_letter.json
│       ├── ieee_a4.json
│       ├── apa_6.json
│       ├── apa_7.json
│       ├── mla_8.json
│       ├── mla_9.json
│       ├── chicago_17.json
│       └── chicago_18.json
│
├── persistence/                  # Session layer
│   ├── ISessionStore
│   ├── SessionStore
│   ├── AutosaveScheduler
│   └── schema/
│       └── session_schema_v1.json
│
├── export/                       # Export layer
│   ├── IExporter
│   ├── DocxExporter
│   └── DocxRenderer
│
├── config/                       # Configuration layer
│   ├── ConfigurationService
│   ├── model.config.json
│   ├── voice_profile.default.json
│   └── reviewer.config.json
│
├── shared/                       # Shared kernel: pure data types, no logic
│   ├── models/
│   │   ├── NotesDocument
│   │   ├── Draft
│   │   ├── FormattedDocument
│   │   └── ReviewerFeedback
│   └── interfaces/                # All the I* contracts, in one place for DI wiring
│
├── tests/
│   ├── unit/
│   │   ├── formatting/            # No model required — fully deterministic
│   │   ├── ai/                    # Mocked IModelClient
│   │   └── persistence/
│   ├── integration/
│   │   └── end_to_end_ieee_pipeline/
│   └── fixtures/
│       └── sample_notes/
│
└── docs/
    ├── architecture.md            # This document
    └── adr/                       # Architecture Decision Records, one file each
```

**Rationale for shape:** every top-level folder is one architectural layer from §2. `shared/` exists specifically to prevent the single most common solo-dev failure mode — circular dependencies — by giving every layer a common, dependency-free vocabulary of data objects to pass between them. `formatting/templates/` being *data files*, not code, is the single decision that makes "six citation-standard/edition combinations" (a Risk in both source documents) tractable in 10 days: adding APA 7th after APA 6th is a new JSON file, not a new class.

---

## 5. Data Flow

```
Research Notes (NotesDocument)
        │
        ▼
   Validation  ──────────────────► [reject / prompt user to complete structure]
        │  (valid)
        ▼
 Prompt Builder  (injects VoiceProfile + section template)
        │
        ▼
   Local AI  (IModelClient → Ollama or llama.cpp, Qwen2.5-3B)
        │
        ▼
  Output Cleaner  (strip artifacts, normalize whitespace, validate non-empty)
        │
        ▼
      Draft  (DraftSection[])
        │
        ├────────────────────────────────────┐
        ▼                                     ▼
  Reviewer Engine (on demand)         Template Engine
   → ReviewerFeedback[]                (applies selected standard/edition,
        │                               produces FormattedDocument)
        │                                     │
        │                                     ▼
        │                              DOCX Export
        │                                     │
        │                                     ▼
        │                              .docx file on disk
        │
        ▼
   Session Save  (NotesDocument + Draft + ReviewerFeedback + TemplateSelection
                   + VoiceProfile → single project file, autosaved throughout)
```

### Step-by-step explanation

1. **Research Notes → Validation:** The `NotesValidator` checks structural completeness (sections present, no empty required fields) *before* any model call is made — this avoids wasting scarce GPU cycles on invalid input, directly serving NFR-5 (reasonable latency on constrained hardware).
2. **Validation → Prompt Builder:** Only validated notes reach the AI layer. The `PromptBuilder` composes one prompt per section (not one giant prompt for the whole paper) — this keeps context windows small enough for a 4GB GPU (NFR-2) and lets each section be regenerated independently without re-running the whole draft.
3. **Prompt Builder → Local AI:** The `IModelClient` sends the prompt to whichever backend is configured (Ollama or llama.cpp) and returns raw text. This is the *only* point in the entire pipeline where the LLM has any influence, and its influence is scoped strictly to prose content.
4. **Local AI → Output Cleaner:** Raw model output is never passed downstream unmodified. Cleaning happens here, once, in one place — not scattered across the formatting or export layers.
5. **Output Cleaner → Draft:** A `Draft` is assembled: an ordered collection of cleaned `DraftSection` objects, each still linked back to its source `NoteSection` for traceability (useful for Reviewer Mode's specificity requirement).
6. **Draft → Reviewer Engine (optional branch):** Reviewer Mode runs against the completed `Draft`, not the raw model, and produces feedback tied to actual section content — this is what satisfies the acceptance criterion that feedback be "specific to that draft's content, not generic boilerplate."
7. **Draft → Template Engine:** The `TemplateEngine` receives the `Draft` and the user's `TemplateSelection` and produces a `FormattedDocument`. No part of this step touches the model; it is 100% deterministic and fully unit-testable in isolation (this is the crux of NFR-4).
8. **FormattedDocument → DOCX Export:** The `DocxExporter` renders the structural tree into a real `.docx` file — margins, fonts, headings, references, captions, all driven by the `FormattedDocument`'s `StyleSheet`, never by free-form LLM text.
9. **Session Save (parallel, continuous):** At every meaningful state transition (notes edited, draft generated, template changed, reviewer run), the `AutosaveScheduler` persists state via `SessionStore`, so a save can happen and a reload can restore *any* point in this flow (FR-6/FR-7), not just the final export.

---

## 6. Component Interaction Diagram

```
 [Presentation]
      │  user actions (events)
      ▼
 [Application/Workflows] ───owns orchestration, no business rules of its own───
      │            │                │              │
      │ calls       │ calls          │ calls         │ calls
      ▼            ▼                ▼              ▼
 [AI Layer]   [Formatting Layer] [Persistence]   [Export Layer]
      │            │                │              │
      └─────┬──────┴───────┬────────┴──────┬───────┘
            │              │               │
            ▼              ▼               ▼
                 [Configuration Layer]
                 (read-only to all above)
```

**Data ownership rules (strict):**

| Data | Owned by | May be read by | May be mutated by |
|---|---|---|---|
| `NotesDocument` | NotesModule | Application, AI Layer (read-only) | NotesModule only |
| `Draft` | AI Layer (created), Application (holds reference) | Formatting Layer, Reviewer Engine, Persistence | AI Layer only (regeneration creates a new Draft, never in-place edits by another layer) |
| `FormattedDocument` | Formatting Layer | Export Layer, Persistence (for caching) | Formatting Layer only |
| `.docx` file | Export Layer | User/filesystem | Export Layer only |
| `SessionState` | Persistence Layer | Application (on load) | Persistence Layer only |
| Config objects | Configuration Layer | Everyone | Configuration Layer only (no runtime mutation by consumers) |

**Module boundaries:** Every arrow above crosses through a public interface (`I*`), never a concrete class reference. This is what makes each layer replaceable and independently testable — a hard requirement given a 10-day solo timeline where there's no time to debug tangled cross-layer dependencies on day 8.

---

## 7. AI Architecture

### Prompt Builder
Composes prompts per section type (Introduction, Literature Review, Methodology, Results, Discussion, Conclusion — as applicable to the notes structure). Each section prompt combines: (1) the relevant `NoteSection` content, (2) the active `VoiceProfile`, (3) a section-specific instructional template (data file, not hardcoded string, so tone/instructions can be tuned without redeploying code).

### Voice Profile
A configuration object (`VoiceProfile`) capturing stylistic parameters — sentence length tendency, formality level, hedging/certainty language, first-person vs. third-person preference, representative style exemplars. Injected into every prompt as instruction + optional few-shot exemplar text. Because voice preservation is inherently approximate (per both source documents' risk sections), the architecture treats it purely as a prompt-engineering concern — there is no separate "voice validation" component pretending to objectively verify style match, since that would overstate what prompting can guarantee.

### Reviewer Mode
A separate `ReviewerEngine` with its own prompt template family, operating on the assembled `Draft` (post-cleaning) rather than notes. This separation exists because reviewer prompts need different instructions (critique framing) than generation prompts (production framing), and conflating them in one prompt builder would create fragile conditional logic.

### Conversation Context
Because each section is generated independently (§5, step 2), "conversation context" per section is deliberately kept small and self-contained rather than accumulating a long multi-turn history — this is both a VRAM-budget decision (NFR-2) and a reliability decision (small 3B models degrade with long context).

### Prompt Templates
Stored as data files (`ai/prompt/SectionPromptTemplates/`), not inline strings in code, so prompt iteration during the 10-day build doesn't require touching orchestration logic.

### Response Validation & Output Cleaning
`OutputCleaner` performs: truncation-boundary repair (drop incomplete trailing sentence), boilerplate/refusal-phrase stripping, whitespace normalization, and a non-empty/minimum-length check that triggers a retry rather than silently accepting an empty section.

### Error Recovery
If `IModelClient.generate()` throws (model not loaded, OOM, timeout), `GenerationWorkflow` catches it, surfaces a clear, actionable error to the Presentation layer, and never lets a failure corrupt the `Draft` object already accumulated for prior sections — partial drafts remain usable.

### Model Configuration
`model.config.json` holds: backend selection (`ollama` | `llamacpp`), model name/path, quantization level, context window size, temperature/top-p, and hard timeout values — all tuned empirically against the 4GB GPU target per the Hardware Risk noted in both source documents.

### Ollama / llama.cpp Integration
Both are implemented strictly behind `IModelClient`. `OllamaClient` talks to a local Ollama daemon over its local HTTP API (still 100% offline — no external network egress); `LlamaCppClient` talks to a local llama.cpp server/binary. Selection is a config value, not a code branch scattered through business logic — the rest of the system is unaware which one is active.

---

## 8. Template Engine

### Design

A single generic `TemplateEngine` operates over **data-driven `TemplateDefinition` files** — one JSON per standard/edition (`ieee_us_letter.json`, `apa_7.json`, etc.), all conforming to one shared schema:

```json
{
  "standard": "APA",
  "edition": "7",
  "pageSize": "US Letter",
  "margins": { "top": "1in", "bottom": "1in", "left": "1in", "right": "1in" },
  "font": { "family": "Times New Roman", "size": 12 },
  "spacing": { "line": "double", "paragraphIndent": "0.5in" },
  "headings": {
    "levels": [
      { "level": 1, "style": "centered-bold" },
      { "level": 2, "style": "left-bold" }
    ]
  },
  "citation": {
    "inTextStyle": "author-date",
    "referenceListTitle": "References",
    "referenceEntryFormat": "{author}. ({year}). {title}. {source}."
  },
  "titlePage": { "required": true, "fields": ["title", "author", "affiliation"] }
}
```

### Why this avoids duplication

- **One `TemplateEngine` class**, not one class per standard. The engine reads whichever `TemplateDefinition` is selected and applies the *same* resolution logic (`LayoutResolver`, `CitationFormatter`) against different data.
- **`CitationFormatter` is a strategy resolved from data**, not a subclass per standard — e.g., `"inTextStyle": "author-date"` vs. `"numeric"` selects a formatting function from a small internal registry, rather than requiring an `APACitationFormatter` class, an `IEEECitationFormatter` class, etc.
- **Common template interface:** every `TemplateDefinition` conforms to one schema (validated on load by `ConfigurationService`), so `TemplateEngine.apply()` never needs standard-specific conditional branches — only data-driven lookups.
- **Editions as sibling files:** APA 6th and APA 7th are two files sharing the same schema shape, differing only in field values (e.g., APA 6th's running-head rules vs. APA 7th's), which is exactly the "depth over breadth, but easy to add more" trade-off the source documents call out as acceptable.

### Adding a new standard/edition in V2

Add a new JSON file conforming to the schema, register it in the template registry (`config/`), done — zero changes to `TemplateEngine`, `CitationFormatter`, or `LayoutResolver` code.

---

## 9. Session Architecture

- **Autosave:** `AutosaveScheduler` triggers on a debounce timer after any state-changing action (note edit, draft generation, template change, reviewer run). Writes go to a temp file first, then atomically replace the prior save file — this prevents a crash mid-write from corrupting the last-known-good session (directly serving the "no data loss" acceptance criterion).
- **Manual Save:** A user-triggered save is the same underlying `SessionStore.save()` call, just invoked explicitly rather than on a timer — no separate code path, so there's only one save routine to get right and test.
- **Session Recovery:** On load, `SessionStore` validates the file against `session_schema_v1.json`. If validation fails (corruption, partial write), it attempts recovery from the last autosave temp file before reporting failure to the user — never silently returning a half-populated state.
- **Project Files:** A single `.scholarforge` file per project (JSON, optionally zipped if binary assets like embedded images are added later) — this keeps the "single-session, single-user, local-only" model (an explicit V1 assumption) simple: no database, no multi-file consistency problem.
- **Draft Versioning:** Out of scope for V1 (not in the PRD's functional requirements), but the schema is versioned (`schema_version` field) from day one specifically so V2 can add a version-history array to the same file format without a breaking migration.
- **Export Metadata:** Each export writes a small metadata record (template used, export timestamp) into the session file, so reopening a project shows what was last exported, without that becoming a separate persistence subsystem.

---

## 10. Export System

The `DocxExporter` takes one input only — the `FormattedDocument` — and never reasons about content, only structure-to-OOXML mapping. This keeps export behavior 100% deterministic and testable without a model in the loop.

| Concern | How it's handled |
|---|---|
| **DOCX generation** | `DocxRenderer` maps the `FormattedDocument` structural tree directly to OOXML via a local, zero-cost DOCX library — no network, no paid service (NFR-6). |
| **Images** | If a `NoteSection` includes an image reference, it flows through untouched from `NotesDocument` → `Draft` → `FormattedDocument` → embedded as a DOCX inline image; the AI layer never touches image content. |
| **Tables** | Table data in notes is preserved as structured data throughout the pipeline (never flattened into prose) and rendered as a native DOCX table. |
| **Captions** | Captions are formatted per the active `TemplateDefinition`'s numbering/labeling rules (e.g., "Figure 1." vs "Fig. 1"), resolved the same way headings are — data-driven, not hardcoded per standard. |
| **References** | The reference list is built entirely by `CitationFormatter` from structured source metadata already present in `NotesDocument.sourceRefs` — the LLM is never asked to "remember" or generate citations, which would be unreliable and is explicitly excluded from V1 scope (no automatic citation lookup). |
| **Margins / Fonts / Spacing / Page Sizes** | Pulled directly from the resolved `StyleSheet` in `FormattedDocument`, one-to-one field mapping into DOCX section/style properties. |
| **Headers / Footers** | Defined per `TemplateDefinition` (e.g., IEEE running headers vs. APA page-number-only headers) and applied by `DocxRenderer` as a discrete rendering step, isolated from body content rendering. |

---

## 11. Configuration System

All configuration lives under `config/` and is loaded once at startup by `ConfigurationService`, which validates each section and exposes typed getters. No other module reads a config file directly.

| Config domain | File | Contains |
|---|---|---|
| **AI settings** | `model.config.json` | Backend choice, model path, context window, temperature, timeouts |
| **Voice profile** | `voice_profile.default.json` (+ user-created overrides) | Style parameters, exemplar snippets |
| **Template selection** | Runtime `TemplateSelection` (chosen per-project, stored in session file), resolved against `formatting/templates/*.json` registry | Active standard + edition + page size |
| **Reviewer options** | `reviewer.config.json` | Feedback categories to check, verbosity level |
| **Export settings** | Part of `model.config.json`'s sibling `export.config.json` (or a section within a unified config) | Output path defaults, DOCX metadata author field |
| **Project settings** | Stored per-project inside the `.scholarforge` session file, not in global config | Project name, associated template, associated voice profile |

Global config vs. per-project config is a deliberate split: settings that are true "installation-wide defaults" (which model, which backend) live in global config; settings that vary "per paper" (which template, which voice profile) travel with the session file so a project stays reproducible even if global defaults later change.

---

## 12. Error Handling Strategy

| Failure | Detection | Handling |
|---|---|---|
| **AI failure (crash, OOM, timeout)** | `IModelClient` call throws or times out per configured limit | Caught at `GenerationWorkflow`; user sees a specific, actionable error ("Model failed to respond — check that Ollama is running"); already-generated sections in the current `Draft` are preserved |
| **Missing model** | `ModelClient` initialization check at startup or first generation call | Clear pre-flight check with a setup-guidance message, rather than a cryptic runtime exception mid-generation |
| **Export failure** | `DocxExporter` catches library-level exceptions (invalid structure, disk write failure) | Reports which section/element failed; the `FormattedDocument` itself is preserved so export can be retried without redoing generation |
| **Corrupt session** | Schema validation fails on `SessionStore.load()` | Falls back to last autosave temp file; if that also fails, reports corruption explicitly rather than loading partial/garbage state |
| **Template mismatch** | `TemplateEngine.apply()` validates the `Draft`'s section structure against the selected `TemplateDefinition`'s required fields (e.g., title page fields) | Surfaces which required field is missing rather than silently omitting it from the DOCX |

**Recovery workflow (general pattern):** every layer fails *loudly and specifically* rather than swallowing errors or guessing — this is a deliberate trade-off favoring debuggability during a tight 10-day solo build over "smart" silent fallback behavior that would be hard to test and hard to trust during a live demo (NFR-7).

---

## 13. Security & Privacy

- **Offline operation:** No layer in this architecture makes an outbound network call in the normal operational path. The only two components that touch a network socket at all (`OllamaClient`, if Ollama's API is accessed over localhost HTTP) communicate exclusively with `localhost`/a local process — never an external host. This satisfies NFR-1/NFR-3 architecturally, not just by policy: there is no HTTP client configured with an external base URL anywhere in the AI, Formatting, Persistence, Export, or Configuration layers.
- **Data ownership:** All `NotesDocument`, `Draft`, and session data lives exclusively in local files owned by the user's account, under a user-chosen project directory. Nothing is ever serialized to a remote endpoint.
- **No telemetry:** No analytics, crash-reporting, or usage-tracking module exists in this architecture. If any is added later, it must be an explicit, separately-approved addition — not something that sneaks in via a bundled library's default behavior.
- **Local storage:** Session files, autosaves, and exported DOCX files all live on local disk, under paths controlled by `ConfigurationService`/user choice.
- **Privacy guarantee (architectural, not just promised):** Because the AI Layer's only external dependency is a local model runtime, and every other layer is pure local computation, "no user data leaves the machine" is a structural property of this design, not a policy statement layered on top of a system that could technically phone home.

---

## 14. Scalability Strategy (V2 Readiness Without Touching V1)

| V2.0 Vision item | How it slots in |
|---|---|
| **LoRA personalization** | New `IModelClient` implementation or a config flag on existing clients pointing at a LoRA-adapted model file — `PromptBuilder`/`GenerationWorkflow` unaware of the change |
| **Fine-tuned academic model** | Same seam as above — swap `model.config.json`'s model path |
| **Citation manager integration** | New `INotesSource` adapter (e.g., `ZoteroImportAdapter`) feeding `NotesDocument.sourceRefs`, upstream of everything else |
| **Research paper library** | New Persistence-layer feature (multi-project index) — additive, doesn't change the single-project session format |
| **PDF export** | New `IExporter` implementation (`PdfExporter`) alongside `DocxExporter`, both behind the same interface |
| **Semantic search** | New capability inside `NotesModule`/Application layer over stored notes — doesn't touch AI, Formatting, or Export layers |
| **Journal templates** | New `TemplateDefinition` data files — zero engine changes, per §8 |
| **Multi-language support** | New `VoiceProfile`/`PromptTemplate` variants + locale-aware `TemplateDefinition` fields — additive data, not new code paths |
| **Cloud synchronization** | New `ISessionStore` implementation (`CloudSessionStore`) alongside the local one, selected by config — the rest of the system already only depends on the `ISessionStore` interface |
| **Plugin architecture** | Falls naturally out of the fact that AI backends, exporters, and template definitions are already registry-driven rather than hardcoded — a plugin system in V2 is largely "expose these existing registries to external code" |

The unifying reason none of this requires touching V1 code: **every extension point in V2's vision maps to an interface (`I*`) that already exists in V1**, because the architecture was built around ports-and-adapters from day one rather than retrofitted later.

---

## 15. Risk Analysis

| Risk | Category | Mitigation |
|---|---|---|
| Qwen2.5-3B produces incoherent or generic prose | Technical / Model quality | Section-by-section generation (smaller, more constrained prompts than whole-paper generation); strong per-section prompt templates; `OutputCleaner` rejects and retries empty/degenerate output |
| 4GB VRAM insufficient for reasonable context + quantization | Performance | Empirical testing early (Day 1–2, not Day 9) of quantization level and max context window; config-driven so values can be tuned without code changes |
| Six standard/edition combinations is a lot of formatting logic for 10 days | Scope / Timeline | Data-driven `TemplateDefinition` schema (§8) turns "new standard" into "new JSON file," not "new class"; PRD explicitly permits depth-over-breadth if time runs short — architecture supports shipping IEEE first and adding others incrementally without refactoring |
| Voice preservation is inherently unverifiable objectively | Product / Testing | Architecture doesn't attempt automated voice-match scoring; treated purely as a manual-review acceptance criterion, avoiding wasted engineering effort on an unsolvable validation problem |
| Solo 10-day builds fail on integration/polish, not core features | Timeline | Strict layer interfaces mean each layer can be unit-tested in isolation *before* integration day, catching interface mismatches early rather than on Day 9; formatting/export layers need zero model access to test, so most of the test suite can run without ever invoking the LLM |
| Maintainability risk from a solo dev under time pressure taking shortcuts | Maintainability | `shared/interfaces` and data-driven templates make "the fast way" and "the right way" the same amount of effort — e.g., adding a template is *faster* as a data file than as hardcoded logic, so the correct pattern is also the path of least resistance |
| Live demo crash (NFR-7) | Deployment / Testing | Loud, specific error handling (§12) surfaces problems during rehearsal rather than producing silent corrupt state that only fails live; autosave means a crash mid-demo doesn't lose the working project |

---

## 16. Testing Architecture

| Test type | Scope | Notes |
|---|---|---|
| **Unit Testing** | `Formatting Layer` (fully, no model needed), `Persistence Layer` (serialization round-trips), `OutputCleaner` (pure string transforms), `CitationFormatter` (data-driven, easy table-based tests per standard/edition) | This is where most test coverage should concentrate — it's cheap to run (no model, no I/O) and covers the layer with the strongest correctness requirement (NFR-4) |
| **Integration Testing** | `GenerationWorkflow` end-to-end against a mocked `IModelClient` (deterministic fake responses) to verify orchestration logic without needing the real model loaded | Also: one real end-to-end run per template standard against the actual local model, to validate the full pipeline at least once |
| **Manual Testing** | Voice-profile fidelity (inherently subjective, per Risk §15), Reviewer Mode feedback specificity, DOCX visual inspection in Word/LibreOffice | These are the acceptance criteria that can't be meaningfully automated |
| **Acceptance Testing** | Each PRD acceptance criterion (§10 of the PRD) mapped to one explicit test case — e.g., "given notes + IEEE selection, produced DOCX has correct margins" is a concrete, checkable test, not just a code review nod | Run before calling any template standard "done" |
| **Regression Testing** | Re-run the full `tests/integration/end_to_end_ieee_pipeline` suite whenever `TemplateEngine`, `DocxExporter`, or shared data models change | Cheap because it's mostly deterministic (mocked model), so it can run frequently, even daily, during the 10-day build |

---

## 17. Development Guidelines

- **Naming conventions:** Interfaces prefixed `I` (`IModelClient`, `ITemplateEngine`); data-only objects suffixed by role (`...Document`, `...Result`, `...Feedback`); no abbreviations that aren't domain-standard (e.g., `DOCX`, `IEEE` are fine; ad hoc shortenings are not).
- **Folder conventions:** One folder per layer at the top level (§4); no cross-layer imports except through `shared/interfaces` and `shared/models`.
- **Documentation standards:** Every public interface gets a one-paragraph docstring describing its contract (inputs, outputs, failure modes) — since a solo developer returning to this code after a break needs the interface docs to *be* the design memory.
- **Commit strategy:** Small, layer-scoped commits (e.g., "formatting: add APA 7th template definition," "ai: add output cleaner truncation repair") — this makes it easy to bisect if the Day 8 integration risk (per both source docs) materializes.
- **Branch strategy:** Given a solo 10-day timeline, a lightweight trunk-based approach is appropriate — short-lived feature branches per module (`feature/template-engine`, `feature/docx-export`), merged to main once that module's unit tests pass, rather than a long-lived branching model that adds overhead without a team to justify it.

---

## 18. Architecture Decision Records (ADR)

### ADR-001: Separate Deterministic Formatting from AI Generation
- **Decision:** Formatting logic and DOCX/template rendering are 100% deterministic code with no LLM involvement; the LLM only ever produces prose content.
- **Reason:** NFR-4 explicitly requires formatting to be independent of LLM variance; this is also the only way to make the formatting layer unit-testable without a model loaded.
- **Alternative considered:** Ask the LLM to directly output formatted text (e.g., "write this in APA format"). Rejected — formatting compliance would then depend on model reliability, which both source documents identify as an inherent risk of a small 3B model.
- **Trade-off:** Slightly more upfront engineering (building a real template engine) versus a much weaker, faster-to-prototype but unreliable approach.

### ADR-002: Data-Driven Template Definitions Instead of Per-Standard Classes
- **Decision:** Each citation standard/edition is a JSON data file conforming to a shared schema, not a subclass of a template base class.
- **Reason:** Avoids duplicated renderer code across six standard/edition combinations; makes adding a new standard a data change, not a code change.
- **Alternative considered:** Class-per-standard (`IEEETemplate extends BaseTemplate`). Rejected — leads to duplicated layout logic across subclasses and violates the "do not duplicate renderer code" constraint.
- **Trade-off:** Requires more upfront schema design effort; pays off immediately given six required combinations.

### ADR-003: `IModelClient` Abstraction over Ollama/llama.cpp
- **Decision:** Both backends implement one interface; backend choice is a config value.
- **Reason:** PRD explicitly allows either backend; hard-coding one would contradict the "modular, extensible" mandate and make backend switching (a likely need given 4GB VRAM constraints) require code changes instead of config changes.
- **Alternative considered:** Pick one backend and hardcode it. Rejected — removes flexibility needed to empirically tune for the 4GB GPU constraint (Hardware Risk in both source docs).
- **Trade-off:** Slightly more integration work upfront (two adapters instead of one) for meaningfully more runtime flexibility.

### ADR-004: Single-Process Monolith, Not Distributed Services
- **Decision:** ScholarForge is one local application process (with the local model runtime as a separate but co-located process), not a client-server or microservice architecture.
- **Reason:** Single-user, single-session, fully offline requirement (explicit V1 assumption) makes distributed architecture pure overhead with no corresponding benefit.
- **Alternative considered:** Split UI and backend into separate services communicating over local sockets. Rejected — adds deployment and testing complexity a solo 10-day build cannot afford, with no scalability benefit for a single local user.
- **Trade-off:** Less "impressive" architecture on paper, but correctly scoped to actual requirements.

### ADR-005: Section-by-Section Generation Instead of Whole-Document Generation
- **Decision:** The AI layer generates one paper section at a time rather than one prompt for the entire paper.
- **Reason:** Keeps prompt/context size within what a 3B model on 4GB VRAM can handle reliably (NFR-2, NFR-5); also allows partial-failure recovery (one bad section doesn't require regenerating the whole draft).
- **Alternative considered:** Single mega-prompt for the whole document. Rejected — high risk of exceeding context limits or producing degraded quality in later sections given typical small-model context degradation.
- **Trade-off:** Slightly more orchestration logic (looping over sections, stitching results) in exchange for reliability on constrained hardware.

### ADR-006: Session State as a Single Local Project File
- **Decision:** All session state (notes, draft, template selection, voice profile reference, reviewer feedback) persists to one versioned local file per project.
- **Reason:** Matches the explicit V1 assumption of single-session, single-user, local-only workflow; avoids the complexity of a database for a single-file-per-project use case.
- **Alternative considered:** A local embedded database (e.g., SQLite). Rejected for V1 as unnecessary complexity — a single-user, single-project-at-a-time tool doesn't need query capability, only serialize/deserialize.
- **Trade-off:** Would need revisiting if V2's "research paper library" (multi-project) becomes a hard requirement — but the schema-versioned file format is designed to make that migration additive, not a rewrite.

---

## 19. Final Technical Summary

ScholarForge V1 is a single-process, offline, layered application built around one non-negotiable seam: **AI generates language; deterministic code generates formatting.** Everything else in this architecture exists to make that seam clean and everything around it swappable.

Concretely: notes flow through validation into a section-by-section prompt/generate/clean loop (`AI Layer`, behind `IModelClient`, backed by Ollama or llama.cpp running Qwen2.5-3B) to produce a `Draft`. The `Draft` is handed to a single, generic `TemplateEngine` that resolves layout and citation rules from data files (`TemplateDefinition` JSON, one per standard/edition) — never from LLM output — producing a `FormattedDocument`. That structural tree is rendered to a real `.docx` by a `DocxExporter` that has no knowledge of, or dependency on, model behavior. In parallel, `SessionManager` continuously autosaves the full working state to a single local project file, and `ReviewerEngine` can run against any completed `Draft` to produce content-specific feedback.

Every layer talks to every other layer through a narrow interface (`I*`) and plain data objects, never shared mutable state — which is what makes each layer unit-testable in isolation (crucial for a solo 10-day build with no time for tangled integration debugging), and what makes every item on the V2.0 roadmap (LoRA, RAG, PDF export, citation manager integration, plugins, cloud sync) an *additive* change behind an existing interface rather than a rearchitecture.

The single sentence a senior engineer picking this up should remember: **if a piece of logic decides what the document says, it lives in the AI layer behind `IModelClient`; if it decides what the document looks like, it lives in the Formatting or Export layer and never touches the model.**
