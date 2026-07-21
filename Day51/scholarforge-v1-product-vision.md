# ScholarForge — Version 1.0 Product Vision

**Tagline:** A privacy-first local AI academic writing workspace that transforms structured research notes into professionally formatted academic papers while preserving the author's own writing style.

---

## 1. Executive Product Summary

ScholarForge is an offline, locally-run writing workspace that converts structured research notes into properly formatted academic papers. It runs a local Qwen2.5-3B model on consumer hardware (4GB GPU), separating deterministic formatting logic from AI-generated prose so that citation styles, margins, and structure stay reliable while the language generation stays flexible. The product exists to solve a narrow, well-defined problem in 10 days: getting from organized notes to a submittable draft, without depending on cloud APIs, subscriptions, or an internet connection.

## 2. Problem Statement

Students and independent researchers spend disproportionate time on two things that have nothing to do with their actual thinking: reformatting their writing to satisfy a citation standard (IEEE, APA, MLA, Chicago), and translating rough notes into publication-style prose. Existing AI writing tools solve part of this, but they either require constant internet access and paid API keys, send research data to third-party servers (a real concern for unpublished or sensitive work), or produce generic AI-sounding text that erases the author's voice. There is no lightweight, offline tool that respects both data privacy and authorial style while handling the mechanical formatting correctly.

## 3. Target Users

**Primary**
- Undergraduate students
- Postgraduate students
- Final-year project students
- Independent researchers

**Secondary**
- Professors
- Technical writers
- Engineers

## 4. Market Gap

Cloud-based AI writing assistants (Jasper, most GPT-wrapper "paper writer" tools) require ongoing internet access and payment, and route the user's research content through third-party servers. Reference managers (Zotero, Mendeley) handle citations but do not generate prose. Word processor plugins handle formatting but have no language generation. No current tool combines local-only inference, multi-standard academic templating, and style preservation in one offline workspace built specifically for the notes-to-draft workflow.

## 5. Unique Value Proposition

ScholarForge is the only offline, zero-cost tool that turns structured notes directly into a formatted academic draft — in the author's own voice, under full local control, with no data ever leaving the machine.

## 6. Version 1.0 Scope

- Structured research notes editor
- Academic paper generation via local Qwen2.5-3B (llama.cpp or Ollama)
- Academic writing voice profile (style preservation through prompt engineering)
- Template engine supporting:
  - IEEE (US Letter, A4)
  - APA (6th, 7th Edition)
  - MLA (8th, 9th Edition)
  - Chicago (17th, 18th Edition)
- DOCX export
- Academic Reviewer Mode
- Session save/load
- Modular architecture separating deterministic formatting from AI-generated content

## 7. Version 1.0 Exclusions

- Fine-tuning or LoRA
- Model training
- Authentication or multi-user support
- Cloud deployment
- Paid APIs
- PDF export
- Automatic citation lookup
- Automatic literature search
- Advanced RAG
- Journal submission workflow
- Collaboration features
- Mobile app

## 8. Version 2.0 Vision

*(Directional only — not designed here)*

- LoRA personalization
- Fine-tuned academic model
- Citation manager integration
- Research paper library
- PDF export
- Semantic search
- Journal templates
- Multi-language support
- Cloud synchronization
- Plugin architecture

## 9. Functional Goals

- User can input structured research notes and receive a generated academic draft
- User can select a citation/formatting standard and edition, and the output conforms to it
- User can save and reload a working session without data loss
- User can invoke Reviewer Mode to get structured feedback on a draft
- User can export a finished draft to DOCX
- Generated prose reflects the configured writing-voice profile rather than generic AI phrasing

## 10. Non-functional Goals

- Fully functional with no internet connection at any point after initial model download
- Runs on a 4GB GPU without crashing or requiring cloud offload
- No user data leaves the local machine
- Formatting rules are deterministic and not subject to LLM variance
- Reasonable generation latency on constrained hardware (target: a section drafts in under a few minutes, not tens of minutes)
- No paid dependencies anywhere in the stack

## 11. Technical Constraints

- Timeline: 10 days, solo developer
- Hardware ceiling: 4GB GPU
- Model: local Qwen2.5-3B only, via llama.cpp or Ollama
- Budget: $0 — no paid APIs, no paid services
- Offline-first: must not architecturally depend on network calls
- Architecture must keep deterministic formatting logic decoupled from LLM output, so template correctness never depends on model behavior

## 12. Success Criteria

- A user can go from structured notes to a correctly formatted DOCX draft in at least one full standard (e.g., IEEE) without manual reformatting
- The system runs end-to-end offline on the target hardware
- Generated text is recognizably distinct from generic, unedited AI output when the voice profile is applied
- Session save/load works reliably without data corruption
- Reviewer Mode produces actionable, non-generic feedback tied to the actual draft content
- The product is stable enough to demo live, cold, without prior warm-up

## 13. Day 10 Definition of Done

By Day 10, ScholarForge must:
- Accept structured research notes as input
- Generate a complete academic paper draft using the local model
- Correctly apply at least one full template standard end-to-end (stretch: more)
- Apply the configured writing-voice profile to generated language
- Export a valid DOCX file
- Support session save and reload
- Run the entire pipeline offline on 4GB GPU hardware
- Survive a live, unrehearsed demo without crashing

Anything not required for this list is out of scope for V1, regardless of how easy it seems to add.

## 14. Product Risks

- **Model quality risk:** Qwen2.5-3B is small; generated academic prose may need heavier prompt scaffolding to sound competent and on-voice.
- **Hardware risk:** 4GB VRAM is tight for a 3B model with reasonable context length; quantization and context limits will need real testing, not assumptions.
- **Scope risk:** Six citation-standard/edition combinations is a lot of deterministic formatting logic to build correctly in 10 days alongside everything else.
- **Style-preservation risk:** "Preserving the author's voice through prompt engineering" is inherently fuzzy and hard to verify objectively; success criteria for this must stay honest about being approximate, not exact.
- **Time risk:** Solo, 10-day builds fail most often from underestimating integration and polish time, not core feature time.

## 15. Product Assumptions

- The user already has structured research notes going in — ScholarForge does not help with research or note-taking itself in V1.
- The user has access to hardware meeting the 4GB GPU minimum.
- "Formatted correctly" means adherence to the mechanical rules of each standard (margins, headings, citation format), not editorial or peer-review quality.
- One well-executed template standard is more valuable at demo time than six shallow ones — depth over breadth is assumed acceptable if time runs short.
- The user is willing to work within a single-session, single-user, local-only workflow for V1.

## 16. Product Vision Statement

ScholarForge exists so that a researcher's own notes and own voice — not a cloud server's guess at either — become a properly formatted academic paper, entirely under the researcher's control.

## 17. One-Page Final Product Summary

**ScholarForge** is an offline academic writing workspace that turns structured research notes into professionally formatted papers using a local Qwen2.5-3B model, with no internet dependency, no paid services, and no data leaving the user's machine. It supports IEEE, APA, MLA, and Chicago formatting across multiple editions, preserves the author's own writing style through prompt engineering, and separates deterministic formatting from AI-generated language so the output stays reliable. Built solo in 10 days on 4GB GPU hardware, V1 deliberately excludes fine-tuning, authentication, collaboration, PDF export, and automated citation lookup — the goal is a polished, working core, not a feature-complete platform. Success is a live, offline demo that takes real notes to a correctly formatted DOCX draft in the author's voice.

---

## ✅ APPROVED VERSION 1.0 SUMMARY

This document — Sections 1 through 17 — is the finalized ScholarForge Version 1.0 product vision, scope, and constraint set. It is the **source of truth** for all subsequent planning artifacts (PRD, technical blueprint, architecture, pitch deck, or implementation plan). Any future document that contradicts or expands the scope defined here should be flagged against this summary before proceeding.
