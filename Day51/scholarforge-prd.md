# ScholarForge — Product Requirements Document (PRD)

**Source of truth:** ScholarForge Version 1.0 Product Vision (Approved)
**Status:** Draft for Build

---

## 1. Executive Summary

ScholarForge is an offline, privacy-first academic writing workspace that converts structured research notes into professionally formatted academic papers using a local Qwen2.5-3B model. It runs entirely on local hardware (4GB GPU), requires no internet connection or paid services after setup, and preserves the author's own writing voice through prompt engineering. This PRD defines the requirements for Version 1.0, to be built solo in 10 days, covering the notes editor, local generation pipeline, multi-standard template engine, DOCX export, Reviewer Mode, and session persistence.

## 2. Vision

ScholarForge exists so that a researcher's own notes and own voice — not a cloud server's guess at either — become a properly formatted academic paper, entirely under the researcher's control.

## 3. Problem Statement

Students and independent researchers lose disproportionate time to two mechanical tasks unrelated to their actual thinking: converting notes into publication-style prose, and reformatting that prose to satisfy a citation standard (IEEE, APA, MLA, Chicago). Existing AI writing tools require internet access and payment, route research content through third-party servers, or produce generic text that erases the author's voice. No current tool combines local-only inference, multi-standard templating, and style preservation in a single offline workspace.

## 4. Goals

### Functional Goals
- Convert structured research notes into a generated academic draft
- Apply a selected citation/formatting standard and edition correctly to output
- Preserve the configured writing-voice profile in generated prose
- Save and reload a working session without data loss
- Provide Reviewer Mode feedback on a draft
- Export a finished draft to DOCX

### Non-functional Goals
- Full functionality with no internet connection after initial model setup
- Stable operation on a 4GB GPU
- No user data leaves the local machine
- Deterministic formatting logic isolated from LLM output variance
- Reasonable generation latency on constrained hardware
- Zero paid dependencies in the stack

## 5. Personas

**Primary**

| Persona | Description | Core Need |
|---|---|---|
| Undergraduate Student | Writing coursework papers, limited formatting experience | Fast, correct formatting + coherent draft from notes |
| Postgraduate Student | Writing thesis chapters or journal-adjacent drafts | Consistent academic voice across a long document |
| Final-Year Project Student | Producing a capstone/dissertation report under deadline pressure | Reliable structure and formatting without manual rework |
| Independent Researcher | Working outside institutional tooling, privacy-conscious | Fully offline, no data leaving their machine |

**Secondary**

| Persona | Description | Core Need |
|---|---|---|
| Professor | Reviewing or drafting academic material | Trustworthy formatting compliance |
| Technical Writer | Producing structured technical documents | Style consistency and template flexibility |
| Engineer | Writing technical/academic reports occasionally | Low-friction tool with no subscription overhead |

## 6. User Stories

1. As a student, I want to enter structured research notes so that the system has organized input to generate from.
2. As a student, I want to select a citation standard and edition (e.g., APA 7th) so that my draft is formatted to my institution's requirements.
3. As a researcher, I want the generated prose to reflect my configured writing voice so that the draft doesn't read as generic AI output.
4. As a user, I want to save my session so that I can return to my draft later without losing progress.
5. As a user, I want to reload a saved session so that I can continue working from where I left off.
6. As a user, I want to run Reviewer Mode on my draft so that I get structured feedback before finalizing it.
7. As a user, I want to export my finished draft to DOCX so that I can submit or share it in a standard format.
8. As a privacy-conscious researcher, I want the entire pipeline to run offline so that my unpublished research never leaves my machine.
9. As a user with limited hardware, I want the application to run within a 4GB GPU budget so that I don't need to upgrade hardware or use a cloud GPU.
10. As a user, I want deterministic formatting (margins, headings, citation structure) so that formatting correctness doesn't depend on model output quality.

## 7. Functional Requirements

| ID | Requirement |
|---|---|
| FR-1 | The system shall provide a structured research notes editor for input capture. |
| FR-2 | The system shall generate academic paper drafts from notes using a local Qwen2.5-3B model via llama.cpp or Ollama. |
| FR-3 | The system shall support template selection across IEEE (US Letter, A4), APA (6th, 7th), MLA (8th, 9th), and Chicago (17th, 18th). |
| FR-4 | The system shall apply the selected template's formatting rules deterministically, independent of LLM output. |
| FR-5 | The system shall apply a configurable academic writing-voice profile to generated content via prompt engineering. |
| FR-6 | The system shall support saving the current session state to local storage. |
| FR-7 | The system shall support loading a previously saved session with no data loss. |
| FR-8 | The system shall provide an Academic Reviewer Mode that generates structured feedback on a draft. |
| FR-9 | The system shall export a completed draft to DOCX format. |
| FR-10 | The system shall operate with a modular architecture separating the deterministic formatting layer from the AI generation layer. |

## 8. Non-Functional Requirements

| ID | Requirement |
|---|---|
| NFR-1 | The system shall function with no internet connection during normal operation (post model-download). |
| NFR-2 | The system shall run on hardware with a 4GB GPU without crashing or requiring cloud offload. |
| NFR-3 | The system shall not transmit user data to any external server or service. |
| NFR-4 | The system shall keep formatting rules deterministic, not subject to LLM output variance. |
| NFR-5 | The system shall complete generation of a document section within a reasonable time on constrained hardware (target: minutes, not tens of minutes). |
| NFR-6 | The system shall have zero paid dependencies (no paid APIs, subscriptions, or licensed services). |
| NFR-7 | The system shall remain stable through a full, unrehearsed live demo without crashing. |

## 9. Architecture Overview

*(High-level only, per Product Bible constraints — no implementation detail)*

ScholarForge follows a modular architecture with a clear boundary between two layers:

- **Deterministic Formatting Layer:** Owns template rules for each citation standard and edition (margins, headings, citation structure, page layout). This layer's output does not depend on the LLM and is independently verifiable/testable.
- **AI Generation Layer:** Owns prose generation from structured notes via the local Qwen2.5-3B model, including application of the writing-voice profile and Reviewer Mode feedback generation.

These two layers communicate through a defined interface so that generated content is populated into the deterministic template structure, rather than the LLM producing formatting decisions directly. Session state (notes, draft progress, selected template) is persisted locally to support save/load. DOCX export operates on the final assembled document, after both layers have contributed their parts.

## 10. Acceptance Criteria

- Given structured notes and a selected template standard, the system produces a complete draft with that standard's formatting correctly applied.
- Given a generated draft, the applied writing-voice profile produces text that is recognizably distinct from unedited generic AI output.
- Given a saved session, reloading it restores notes, draft state, and template selection without data loss or corruption.
- Given a completed draft, Reviewer Mode returns feedback specific to that draft's content, not generic boilerplate.
- Given a completed draft, DOCX export produces a valid, openable file conforming to the selected template.
- The entire workflow — notes input through DOCX export — completes with no network connection active.
- The application runs and completes a full demo scenario on a machine with a 4GB GPU without crashing.

## 11. Success Metrics

- At least one full template standard (e.g., IEEE) works end-to-end with zero manual reformatting required post-export.
- 100% of the generation-to-export pipeline runs offline with no network calls.
- Session save/load succeeds with no data loss across repeated test cycles.
- Reviewer Mode output is judged as content-specific (not generic) in manual review of test drafts.
- The system survives a live, cold-start demo without failure.

## 12. Risks

- **Model quality risk:** Qwen2.5-3B's small size may limit the coherence and sophistication of generated academic prose without strong prompt scaffolding.
- **Hardware risk:** 4GB VRAM constrains context length and quantization choices; needs empirical validation, not assumption.
- **Scope risk:** Six citation-standard/edition combinations represent significant deterministic formatting logic to complete correctly within 10 days.
- **Style-preservation risk:** Voice preservation via prompting is inherently approximate and difficult to verify objectively.
- **Timeline risk:** Solo 10-day builds most often fail on integration and polish time rather than core feature time.

## 13. Assumptions

- Users arrive with structured research notes already prepared; ScholarForge does not assist with research or note-taking itself.
- Users have hardware meeting the 4GB GPU minimum.
- "Correct formatting" refers to mechanical compliance with standard rules, not editorial or peer-review quality.
- Depth on one template standard is acceptable over shallow coverage of all six if time is constrained.
- A single-session, single-user, fully local workflow is acceptable for V1.

## 14. Glossary

| Term | Definition |
|---|---|
| **Qwen2.5-3B** | A 3-billion-parameter local language model used for on-device text generation. |
| **llama.cpp / Ollama** | Local inference runtimes used to run the Qwen2.5-3B model on-device without cloud dependency. |
| **Deterministic Formatting Layer** | The part of the system responsible for citation/template rules, which does not depend on LLM output. |
| **Writing-Voice Profile** | A configured set of stylistic characteristics applied via prompt engineering to make generated text match the author's style. |
| **Reviewer Mode** | A feature that generates structured feedback on a draft, distinct from draft generation itself. |
| **Session** | The saved state of a user's notes, draft progress, and template selection. |
| **Template Engine** | The component responsible for applying a selected citation standard/edition's formatting rules. |
| **IEEE / APA / MLA / Chicago** | Supported academic citation and formatting standards, each with specific supported editions per Section 3 of the Product Bible. |

---

This PRD reflects only what is defined in the approved ScholarForge Version 1.0 Product Bible. No scope, feature, or constraint has been added, removed, or reinterpreted.
