# ScholarForge — Localhost API Specification (V1.0)

**Source of truth:** ScholarForge V1.0 Product Vision + PRD + Software Architecture Document + Implementation Blueprint
**Author role:** Principal Software Architect
**Status:** Implementation-ready API design — gates Day 3 development
**Scope discipline:** Every endpoint maps to a PRD functional requirement (FR-1 through FR-10). No endpoint is added that isn't traceable to an approved requirement.

---

## 1. API Design Principles

### 1.1 Conventions

| Convention | Choice | Rationale |
|---|---|---|
| **Transport** | HTTP/1.1 | localhost only — no HTTP/2 push needed, no TLS needed for local mode |
| **Format** | JSON for all request/response bodies | Matches the file-based storage format; no binary in V1 |
| **Auth** | None | Single-user, local-only — auth is V1 exclusion (Vision §7) |
| **CORS** | Permissive for Vercel origin + localhost | Required only for secondary deployment mode |
| **Errors** | RFC 7807 Problem Details (simplified) | Consistent, machine-parseable error shape |
| **Versioning** | `/api/v1/...` prefix | Future-proofing without complexity |
| **State** | Stateless backend | ADR-008 — each request contains full input; no server-side sessions |

### 1.2 Base URL

```
http://localhost:3001/api/v1
```

All endpoints below are relative to this base.

### 1.3 Standard Error Response Shape

Every error response follows this shape:

```json
{
  "error": {
    "code": "NOTES_INVALID",
    "message": "Notes validation failed: Section 2 has no bullets",
    "details": [
      { "field": "sections[1].bullets", "issue": "must have at least 1 element" }
    ]
  }
}
```

| Field | Type | Purpose |
|---|---|---|
| `error.code` | string (SCREAMING_SNAKE) | Machine-parseable error code for frontend branching |
| `error.message` | string | Human-readable single-line summary |
| `error.details` | array (optional) | Field-level specifics for form validation |

### 1.4 Standard HTTP Status Codes Used

| Code | Meaning | When |
|---|---|---|
| 200 | OK | Successful GET, successful mutation with response body |
| 201 | Created | Successful POST that creates a resource (e.g., session save creates file) |
| 204 | No Content | Successful mutation with no response body |
| 400 | Bad Request | Validation failure (zod schema mismatch, invalid input) |
| 404 | Not Found | Resource doesn't exist (e.g., session file not found on load) |
| 408 | Request Timeout | Model call exceeded configured timeout |
| 422 | Unprocessable Entity | Input is valid JSON but semantically wrong (e.g., template selection references non-existent standard) |
| 500 | Internal Server Error | Unexpected backend failure (should be rare — logged with stack trace) |
| 503 | Service Unavailable | Model backend (Ollama/llama.cpp) not reachable |

---

## 2. Health & Configuration Endpoints

### 2.1 `GET /health`

**Purpose:** Backend liveness check. Used by the frontend on app launch to confirm the Express server is running.

**Request:** No body, no params.

**Response (200):**
```json
{
  "status": "ok",
  "version": "1.0.0",
  "uptime": 12345,
  "modelBackend": {
    "configured": "ollama",
    "reachable": true,
    "modelName": "qwen2.5:3b-instruct-q4_K_M"
  }
}
```

**Response (503)** — backend up but model unreachable:
```json
{
  "status": "degraded",
  "version": "1.0.0",
  "uptime": 12345,
  "modelBackend": {
    "configured": "ollama",
    "reachable": false,
    "error": "ECONNREFUSED — is Ollama running on localhost:11434?"
  }
}
```

**Validation:** None.
**Error handling:** Never returns 5xx for itself — if the endpoint can respond at all, it returns 200 or 503 based on model reachability.

---

### 2.2 `GET /config/model`

**Purpose:** Retrieve the current model configuration (for display in the UI settings panel). Does NOT include secrets (there are none in V1, but the pattern is established).

**Request:** No body.

**Response (200):**
```json
{
  "backend": "ollama",
  "modelName": "qwen2.5:3b-instruct-q4_K_M",
  "quantization": "Q4_K_M",
  "contextWindow": 4096,
  "temperature": 0.7,
  "topP": 0.9,
  "timeoutMs": 30000
}
```

**Validation:** None (read-only).
**Error handling:** 500 if config file is missing/corrupt (should never happen post-startup validation).

---

### 2.3 `GET /config/templates`

**Purpose:** List all available template definitions (for the Template Picker UI). Returns metadata only, not full template content.

**Request:** No body.

**Response (200):**
```json
{
  "templates": [
    {
      "id": "ieee_us_letter",
      "standard": "IEEE",
      "edition": "1",
      "pageSize": "US Letter",
      "displayName": "IEEE (US Letter)"
    },
    {
      "id": "apa_7",
      "standard": "APA",
      "edition": "7",
      "pageSize": null,
      "displayName": "APA 7th Edition"
    }
  ]
}
```

**Validation:** None.
**Error handling:** 500 if template registry fails to load (config validation should have caught this at startup).

---

### 2.4 `GET /config/voice-profiles`

**Purpose:** List available voice profiles (default + any user-created).

**Request:** No body.

**Response (200):**
```json
{
  "profiles": [
    {
      "profileId": "default",
      "profileName": "Default Academic",
      "formality": "formal",
      "sentenceLengthTendency": "varied",
      "personPreference": "third_person",
      "hedgingLevel": "moderate"
    },
    {
      "profileId": "user_concise_1",
      "profileName": "My Concise Style",
      "formality": "neutral",
      "sentenceLengthTendency": "short",
      "personPreference": "first_person",
      "hedgingLevel": "direct"
    }
  ]
}
```

**Validation:** None.
**Error handling:** 500 if voice profile directory is unreadable.

---

## 3. Notes Endpoints (FR-1)

### 3.1 `POST /notes/validate`

**Purpose:** Validate a `NotesDocument` before generation. Returns specific, field-level issues rather than a boolean. Maps to `NotesValidator.validate()` in the Application Layer.

**Request body:**
```json
{
  "notesDocument": {
    "sections": [
      {
        "id": "sec_1",
        "heading": "Introduction",
        "bullets": ["Context of the research problem"],
        "sourceRefs": [],
        "sectionType": "introduction"
      }
    ]
  }
}
```

**Response (200)** — valid:
```json
{
  "valid": true,
  "issues": []
}
```

**Response (200)** — invalid (note: 200, not 400 — the request itself is valid; the notes content is invalid):
```json
{
  "valid": false,
  "issues": [
    {
      "sectionId": "sec_2",
      "field": "bullets",
      "issue": "must have at least 1 element"
    },
    {
      "sectionId": "sec_3",
      "field": "heading",
      "issue": "must be non-empty"
    }
  ]
}
```

**Validation:** Request body validated by zod schema (`NotesDocumentSchema`). 400 if the request body itself is malformed (not a valid `NotesDocument` shape).
**Error handling:** 400 for malformed request; 200 with `valid: false` for semantically invalid notes content.

---

## 4. Generation Endpoints (FR-2, FR-5)

### 4.1 `POST /generate/draft`

**Purpose:** Generate a complete `Draft` from a validated `NotesDocument`, using the configured voice profile and model. This is the core AI operation. Maps to `GenerationWorkflow.generateDraft()`.

**Request body:**
```json
{
  "notesDocument": { "...": "NotesDocument shape" },
  "voiceProfile": { "...": "VoiceProfile shape" },
  "modelConfig": {
    "temperature": 0.7,
    "topP": 0.9
  }
}
```

**Response (200)** — success (possibly with partial failures):
```json
{
  "draft": {
    "sections": [
      {
        "id": "draft_sec_1",
        "sourceSectionId": "sec_1",
        "heading": "Introduction",
        "content": "The rapid evolution of...",
        "generationMetadata": {
          "promptTokensEstimate": 412,
          "responseTimeMs": 3421,
          "retries": 0
        }
      }
    ],
    "failedSectionIds": ["sec_3"],
    "generatedAt": "2024-01-15T10:30:00Z",
    "modelConfigSnapshot": {
      "backend": "ollama",
      "modelName": "qwen2.5:3b-instruct-q4_K_M",
      "quantization": "Q4_K_M",
      "temperature": 0.7,
      "topP": 0.9
    }
  }
}
```

**Response (503)** — model backend unreachable:
```json
{
  "error": {
    "code": "MODEL_UNREACHABLE",
    "message": "Model backend is not reachable. Is Ollama running on localhost:11434?",
    "details": null
  }
}
```

**Response (408)** — model call timed out:
```json
{
  "error": {
    "code": "MODEL_TIMEOUT",
    "message": "Model call exceeded the 30000ms timeout for section 'Introduction'.",
    "details": { "sectionId": "sec_1", "timeoutMs": 30000 }
  }
}
```

**Validation:**
- Request body validated by zod (`GenerateDraftRequestSchema`)
- `notesDocument` is run through `NotesValidator` first — 422 if invalid notes
- `voiceProfile` must reference an existing profile ID or be a full inline profile — 422 if neither

**Error handling:**
- 400: malformed request body
- 422: valid JSON but semantically invalid (notes fail validation, voice profile not found)
- 503: model backend unreachable (pre-flight check failed)
- 408: model call timed out (per-section timeout)
- 200 with `failedSectionIds`: partial success — some sections failed but draft was still assembled

**Notes:**
- This endpoint is **long-running** (section-by-section generation can take 30s–3min depending on hardware). Frontend should show a progress indicator.
- For streaming progress (section-by-section completion), see §4.2 below.
- The endpoint is **idempotent in intent** — calling it twice with the same input produces a new Draft (different `id`s), not an error. The caller decides which Draft to keep.

---

### 4.2 `POST /generate/draft/stream` (SSE variant)

**Purpose:** Same as §4.1, but streams progress as Server-Sent Events so the frontend can show per-section completion in real time.

**Request body:** Same as §4.1.

**Response:** `text/event-stream` — one event per section completed, plus a final `done` event.

**Event: `section_started`**
```
event: section_started
data: {"sectionId": "sec_1", "heading": "Introduction"}
```

**Event: `section_completed`**
```
event: section_completed
data: {"sectionId": "sec_1", "draftSection": {"id": "draft_sec_1", "content": "The rapid evolution of..."}}
```

**Event: `section_failed`**
```
event: section_failed
data: {"sectionId": "sec_3", "error": "MODEL_TIMEOUT", "message": "Section timed out after 30000ms"}
```

**Event: `done`**
```
event: done
data: {"draft": {"...": "full Draft object"}, "failedSectionIds": ["sec_3"]}
```

**Event: `error`** (terminal — generation aborted)
```
event: error
data: {"code": "MODEL_UNREACHABLE", "message": "Ollama not running"}
```

**Validation:** Same as §4.1 — validation errors are sent as the first `error` event.
**Error handling:** Same codes as §4.1, delivered as SSE `error` events.

---

### 4.3 `POST /generate/review` (FR-8)

**Purpose:** Run Reviewer Mode on a completed `Draft`. Returns structured, content-specific feedback per section. Maps to `ReviewerEngine.review()`.

**Request body:**
```json
{
  "draft": { "...": "Draft shape" },
  "reviewerConfig": {
    "categories": ["clarity", "argument", "evidence"],
    "verbosity": "normal",
    "maxFeedbackPerSection": 5
  }
}
```

**Response (200):**
```json
{
  "feedback": [
    {
      "sectionId": "draft_sec_1",
      "issue": "The opening paragraph makes a broad claim about 'rapid evolution' without citing a source.",
      "suggestion": "Add an in-text citation to a specific source from the sourceRefs, or rephrase as 'has been characterized as rapid (Author, Year)'.",
      "severity": "medium",
      "category": "evidence"
    }
  ],
  "reviewedAt": "2024-01-15T10:35:00Z",
  "reviewedDraftId": "draft_abc123"
}
```

**Response (503):** Same as §4.1 — model unreachable.
**Response (408):** Same as §4.1 — per-section timeout during review.

**Validation:**
- `draft` must be a valid `Draft` shape — 400 if malformed
- `reviewerConfig.categories` must be a subset of allowed categories — 422 if invalid

**Error handling:** Same pattern as generation. Reviewer Mode is a separate AI pipeline (critique-framed, not production-framed) but reuses `IModelClient` and `OutputCleaner`.

---

## 5. Formatting Endpoints (FR-3, FR-4, FR-10)

### 5.1 `POST /format/apply`

**Purpose:** Apply a selected template's deterministic formatting rules to a `Draft`, producing a `FormattedDocument`. **No AI involvement** — this is pure deterministic code. Maps to `TemplateEngine.apply()`.

**Request body:**
```json
{
  "draft": { "...": "Draft shape" },
  "templateSelection": {
    "standard": "IEEE",
    "edition": "1",
    "pageSize": "US Letter"
  }
}
```

**Response (200):**
```json
{
  "formattedDocument": {
    "sections": [
      {
        "sourceDraftSectionId": "draft_sec_1",
        "heading": "Introduction",
        "headingLevel": 1,
        "content": "The rapid evolution of..."
      }
    ],
    "styleSheet": {
      "pageSize": "US Letter",
      "margins": { "top": "0.75in", "bottom": "1in", "left": "0.625in", "right": "0.625in" },
      "font": { "family": "Times New Roman", "size": 10 },
      "spacing": { "line": "single", "paragraphIndent": "0.2in" }
    },
    "references": [
      {
        "number": 1,
        "inTextCitation": "[1]",
        "referenceListEntry": "[1] Smith, \"AI in Education,\" Journal of X, 2023.",
        "sourceRefId": "ref_1"
      }
    ],
    "templateSelection": {
      "standard": "IEEE",
      "edition": "1",
      "pageSize": "US Letter"
    }
  }
}
```

**Response (422)** — template mismatch (Draft is missing a required field for this template):
```json
{
  "error": {
    "code": "TEMPLATE_MISMATCH",
    "message": "Draft is missing required field for IEEE: titlePage.title",
    "details": { "missingFields": ["titlePage.title", "titlePage.author"] }
  }
}
```

**Validation:**
- `draft` validated by zod — 400 if malformed
- `templateSelection.standard` + `edition` + `pageSize` must map to a registered template — 422 if not found
- `TemplateEngine.validate()` runs against the Draft — 422 with `TEMPLATE_MISMATCH` if required fields are missing

**Error handling:**
- 400: malformed request body
- 422: template not found OR draft missing required template fields
- 500: unexpected formatting engine error (should never happen — deterministic code, fully unit-tested)

**Notes:**
- This endpoint is **fast** (deterministic, no model call) — typically <100ms.
- This endpoint is **idempotent** — same input always produces same output.

---

### 5.2 `GET /format/templates/{templateId}`

**Purpose:** Retrieve the full `TemplateDefinition` for a specific template (for display/debugging in the UI).

**Path params:**
- `templateId` — e.g., `ieee_us_letter`, `apa_7`

**Response (200):**
```json
{
  "template": {
    "standard": "IEEE",
    "edition": "1",
    "pageSize": "US Letter",
    "margins": { "top": "0.75in", "...": "..." },
    "font": { "family": "Times New Roman", "size": 10 },
    "spacing": { "line": "single", "paragraphIndent": "0.2in" },
    "headings": { "levels": ["..."] },
    "citation": { "inTextStyle": "numeric", "referenceListTitle": "References", "referenceEntryFormat": "..." },
    "titlePage": { "required": false }
  }
}
```

**Response (404):** Template ID not found.

**Validation:** None.
**Error handling:** 404 for unknown template ID.

---

## 6. Export Endpoints (FR-9)

### 6.1 `POST /export/docx`

**Purpose:** Export a `FormattedDocument` to a `.docx` file on the local filesystem. Maps to `DocxExporter.export()`.

**Request body:**
```json
{
  "formattedDocument": { "...": "FormattedDocument shape" },
  "outputPath": "/Users/devpal/Documents/my_paper.docx",
  "metadata": {
    "author": "Devpal Singh Anand",
    "title": "AI in Education: A Review"
  }
}
```

**Response (200):**
```json
{
  "filePath": "/Users/devpal/Documents/my_paper.docx",
  "sizeBytes": 45231,
  "exportedAt": "2024-01-15T10:40:00Z"
}
```

**Response (400)** — invalid output path (not absolute, no write permission):
```json
{
  "error": {
    "code": "INVALID_OUTPUT_PATH",
    "message": "Output path must be absolute. Received: 'my_paper.docx'",
    "details": null
  }
}
```

**Response (403)** — permission denied:
```json
{
  "error": {
    "code": "WRITE_PERMISSION_DENIED",
    "message": "Cannot write to /Users/anotheruser/Documents/ — permission denied.",
    "details": null
  }
}
```

**Response (422)** — FormattedDocument is structurally invalid:
```json
{
  "error": {
    "code": "INVALID_FORMATTED_DOCUMENT",
    "message": "FormattedDocument is missing styleSheet.margins.top.",
    "details": { "missingField": "styleSheet.margins.top" }
  }
}
```

**Validation:**
- `formattedDocument` validated by zod — 400 if malformed
- `outputPath` must be absolute, must end in `.docx`, parent directory must exist — 422 if invalid
- Pre-flight check: can the process write to the parent directory? — 403 if not

**Error handling:**
- 400: malformed request body
- 422: valid JSON but invalid content (bad path, missing FormattedDocument fields)
- 403: filesystem permission denied
- 500: DOCX library throws (malformed OOXML) — should never happen with a valid FormattedDocument, but if it does, the FormattedDocument is preserved for retry

**Notes:**
- This endpoint is **fast** (typically <500ms — no model, no network).
- The exported file's validity is checked by reopening it programmatically (the `docx` library can parse its own output) before returning success.

---

## 7. Session Endpoints (FR-6, FR-7)

### 7.1 `POST /session/save`

**Purpose:** Save the full `SessionState` to a `.scholarforge` file. Atomic write (temp + rename). Used by both manual "Save" and the frontend's debounced autosave. Maps to `SessionStore.save()`.

**Request body:**
```json
{
  "sessionState": { "...": "full SessionState shape" },
  "projectFilePath": "/Users/devpal/Documents/my_project.scholarforge"
}
```

**Response (201):**
```json
{
  "saved": true,
  "projectFilePath": "/Users/devpal/Documents/my_project.scholarforge",
  "savedAt": "2024-01-15T10:45:00Z",
  "sizeBytes": 12453
}
```

**Response (422)** — session state fails schema validation:
```json
{
  "error": {
    "code": "INVALID_SESSION_STATE",
    "message": "SessionState failed schema validation: schemaVersion is required.",
    "details": [{ "field": "schemaVersion", "issue": "required" }]
  }
}
```

**Validation:**
- `sessionState` validated by `session_schema_v1` (zod) — 422 if invalid
- `projectFilePath` must be absolute, must end in `.scholarforge` — 422 if invalid
- `schemaVersion` must equal `1` (current) — 422 if different (future versions handle migration)

**Error handling:**
- 400: malformed request body
- 422: valid JSON but invalid session state or path
- 403: permission denied on the target directory
- 500: atomic write failed (rename failed — extremely rare on POSIX)

**Notes:**
- This endpoint is **idempotent** — saving the same state to the same path twice produces the same file.
- The `.bak` file is updated on every successful save (see SCHEMA.md §2.2).

---

### 7.2 `POST /session/load`

**Purpose:** Load a `.scholarforge` file into a `SessionState`. Includes corruption recovery (falls back to `.bak` if main file is corrupt). Maps to `SessionStore.load()`.

**Request body:**
```json
{
  "projectFilePath": "/Users/devpal/Documents/my_project.scholarforge"
}
```

**Response (200):**
```json
{
  "sessionState": { "...": "full SessionState shape" },
  "loadedFrom": "primary",
  "loadedAt": "2024-01-15T10:50:00Z"
}
```

**Response (200)** — recovered from backup:
```json
{
  "sessionState": { "...": "full SessionState shape" },
  "loadedFrom": "backup",
  "warning": "Primary file was corrupt — loaded from backup. Some recent changes may be lost.",
  "loadedAt": "2024-01-15T10:50:00Z"
}
```

**Response (404)** — file not found:
```json
{
  "error": {
    "code": "FILE_NOT_FOUND",
    "message": "Project file not found: /Users/devpal/Documents/missing.scholarforge",
    "details": null
  }
}
```

**Response (422)** — schema version newer than app supports:
```json
{
  "error": {
    "code": "UNSUPPORTED_SCHEMA_VERSION",
    "message": "This project was created by a newer version of ScholarForge (schemaVersion 2). Please update the app.",
    "details": { "fileSchemaVersion": 2, "appSupportedVersion": 1 }
  }
}
```

**Response (500)** — both primary and backup corrupt:
```json
{
  "error": {
    "code": "CORRUPT_PROJECT_FILE",
    "message": "Project file is corrupt and the backup is also corrupt. The file may have been damaged by an external program.",
    "details": { "primaryError": "Unexpected token < in JSON", "backupError": "Unexpected token < in JSON" }
  }
}
```

**Validation:**
- `projectFilePath` must be absolute, must end in `.scholarforge` — 422 if invalid
- File must exist — 404 if not
- File must be valid JSON conforming to `session_schema_v1` — falls back to backup, then 500 if both corrupt
- `schemaVersion` must be ≤ app's supported version — 422 if higher

**Error handling:** See response codes above. The recovery flow is non-blocking — if the primary file is corrupt, the endpoint automatically tries the backup and returns 200 with a `warning` field, not an error.

---

### 7.3 `GET /session/recent`

**Purpose:** List recently opened project files (for the "Open Recent" menu in the UI). Stored in the global config directory, not in any project file.

**Request:** No body.

**Response (200):**
```json
{
  "recent": [
    {
      "path": "/Users/devpal/Documents/my_project.scholarforge",
      "projectName": "My Capstone Paper",
      "lastOpened": "2024-01-15T10:50:00Z"
    },
    {
      "path": "/Users/devpal/Documents/another.scholarforge",
      "projectName": "Literature Review Draft",
      "lastOpened": "2024-01-14T15:20:00Z"
    }
  ]
}
```

**Validation:** None.
**Error handling:** 500 if the recent-projects file is corrupt (should self-heal by returning an empty list and overwriting the corrupt file).

**Notes:**
- Maximum 10 entries — oldest entries are evicted.
- Entries pointing to files that no longer exist are filtered out on response.

---

## 8. Endpoint Summary Table

| Method | Path | Purpose | Maps to FR | Maps to Workflow |
|---|---|---|---|---|
| GET | `/health` | Backend liveness + model reachability | (NFR-7) | — |
| GET | `/config/model` | Get model config | (NFR-6) | `ConfigurationService` |
| GET | `/config/templates` | List available templates | FR-3 | `ConfigurationService` |
| GET | `/config/voice-profiles` | List voice profiles | FR-5 | `ConfigurationService` |
| POST | `/notes/validate` | Validate notes structure | FR-1 | `NotesValidator` |
| POST | `/generate/draft` | Generate full Draft | FR-2, FR-5 | `GenerationWorkflow` |
| POST | `/generate/draft/stream` | Generate Draft with SSE progress | FR-2, FR-5 | `GenerationWorkflow` |
| POST | `/generate/review` | Run Reviewer Mode | FR-8 | `ReviewWorkflow` / `ReviewerEngine` |
| POST | `/format/apply` | Apply template formatting | FR-3, FR-4, FR-10 | `TemplateEngine` |
| GET | `/format/templates/{id}` | Get full template definition | FR-3 | `ConfigurationService` |
| POST | `/export/docx` | Export to DOCX | FR-9 | `ExportWorkflow` / `DocxExporter` |
| POST | `/session/save` | Save session state | FR-6 | `SessionWorkflow` / `SessionStore` |
| POST | `/session/load` | Load session state | FR-7 | `SessionWorkflow` / `SessionStore` |
| GET | `/session/recent` | List recent projects | (UX) | `ConfigurationService` |

**Total: 14 endpoints.** Every endpoint is traceable to a PRD functional requirement or an explicit non-functional requirement.

---

## 9. CORS Configuration

```javascript
// Express CORS setup — allows Vercel origin + localhost
app.use(cors({
  origin: [
    'http://localhost:3000',           // Next.js dev mode
    'https://scholarforge.vercel.app', // Vercel production (secondary mode)
    // Add preview URLs dynamically if needed
  ],
  credentials: false,                  // No auth in V1
  methods: ['GET', 'POST'],
  allowedHeaders: ['Content-Type'],
}));
```

**Notes:**
- `credentials: false` because there's no auth in V1.
- Only `GET` and `POST` are needed — no `PUT`/`PATCH`/`DELETE` because the backend is stateless and all mutations are full-state replacements.
- Vercel preview URLs (`*.vercel.app`) can be allowed via a regex if needed, but the production URL should suffice for V1.

---

## 10. Rate Limiting & Timeouts

| Concern | Strategy |
|---|---|
| **Generation timeout** | Per-section `timeoutMs` from `model.config.json` (default 30s). Enforced by aborting the underlying HTTP call to Ollama (using `AbortController`), not by a bare timer race. |
| **Export timeout** | 60s hard limit (export is fast, but a pathological FormattedDocument could in theory be large). |
| **Session save timeout** | 10s (file I/O should be near-instant). |
| **Rate limiting** | None in V1. Single-user, local-only — no abuse vector. If V2 adds multi-user, revisit. |
| **Concurrent requests** | Express handles concurrent requests naturally (async I/O). Generation requests are CPU-bound on the model side — the model backend (Ollama) serializes internally, so concurrent generation requests just queue at the model layer. No need for explicit queuing in Express. |

---

## 11. API Versioning Strategy

- All V1 endpoints are under `/api/v1/`.
- V2 will add `/api/v2/` for breaking changes, with V1 endpoints maintained for one minor version to allow migration.
- Non-breaking additions (new endpoints, new optional fields) are added under `/api/v1/` without a version bump.
- The `schemaVersion` field in `SessionState` is independent of the API version — it tracks the file format, not the API contract.

---

## 12. References

| Document | Section |
|---|---|
| Software Architecture Document | §3 (Module breakdown), §5 (Data flow), §12 (Error handling) |
| PRD | §7 (Functional Requirements), §8 (Non-Functional Requirements) |
| This document | `Day52/API.md` |
| ARCHITECTURE.md | §5 (Request lifecycle), §8 (ADR-008 — stateless backend) |
| SCHEMA.md | §3 (Data models — define the request/response shapes) |
| PROJECT-STRUCTURE.md | §3 (Express route file locations) |
