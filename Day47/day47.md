# Day 47 – Content Intelligence Studio

## Overview

A single-file HTML application called "Content Intelligence Studio" — a production-quality AI content consultant that runs a genuine multi-reviewer analysis pipeline. The app guides the user through a 5-question interview to configure the content review context (content type, platform, goal, upload type, review intensity), then dynamically selects 3–6 specialized AI reviewers based on the answers. Every reviewer call is a live API request that returns a structured natural-language review (score, reasoning, strengths, weaknesses, missed opportunities, platform recommendations, improved version, alternative hooks, alternative titles). A final executive report synthesizes all reviewer outputs into one executive-ready brief.

The problem it addresses is that most "AI content review" tools produce a single generic score and a few bullet points. This app does the opposite: it deploys multiple specialized reviewers — each with a production-quality system prompt tuned to its domain (Content Strategy, Platform Growth, Audience Psychology, Engagement Optimization, Visual Design, Conversion) — and lets each one reason independently before synthesizing. The educational objective is understanding how to build a provider-agnostic, multi-agent AI pipeline with natural-language section-marker parsing (no fragile JSON forcing), dynamic reviewer selection, and a real backend proxy that works out-of-the-box with no external API key.

---

## Prompt Template

The following prompt was used to generate Content Intelligence Studio:

```text
You are an elite AI Systems Architect, Agentic AI Researcher, Prompt Engineer, Workflow Automation Engineer, Senior Frontend Engineer, UX Architect, Information Visualization Designer, and Product Designer.

Your mission is to build a production-quality AI-powered content consultant platform.

PHASE 1 — INTERVIEW
Ask ONLY one multiple-choice question at a time. Never ask multiple questions together. Every question must include professional options + "Other". Do not generate any code until the interview is completely finished.

Interview Flow:
Q1: What type of content are we optimizing? (Blog post, Social media caption, Video script, Newsletter, Landing page copy, Ad copy, Product description, Email campaign, Other)
Q2: Which platform will this content live on? (LinkedIn, Twitter/X, Instagram, YouTube, Medium, Substack, TikTok, Website/Blog, Email, Other)
Q3: What is the primary goal of this content? (Engagement, Lead generation, Brand awareness, Thought leadership, Sales conversion, Audience growth, Education, Other)
Q4: How will you provide the content? (Text only, Image/Screenshot, Other)
Q5: How deep should the review go? (Quick scan = 3 reviewers, Standard review = 4–5 reviewers, Deep analysis = all relevant reviewers, Other)

PHASE 2 — APPLICATION
Build a SINGLE self-contained HTML file. No React, Vue, Tailwind, Bootstrap, CDN, or external libraries. Only HTML, CSS, Vanilla JavaScript. Everything must work offline (except the AI API calls).

REAL MULTI-REVIEWER PIPELINE:
The analysis MUST be a genuine sequential loop that runs each selected reviewer one at a time. Each reviewer is a live API call. Never fake scores, reasoning, or improvements. Each reviewer outputs natural-language sections using ## MARKER: headers (SCORE, REASONING, STRENGTHS, WEAKNESSES, MISSED OPPORTUNITIES, PLATFORM RECOMMENDATIONS, IMPROVED VERSION, ALTERNATIVE HOOKS, ALTERNATIVE TITLES) which are parsed by section markers — no JSON forcing.

DYNAMIC REVIEWER SELECTION:
Based on the interview answers, select 3–6 reviewers from: Content Strategy Reviewer, Platform Growth Reviewer, Audience Psychology Reviewer, Engagement Optimization Reviewer, Visual Design Reviewer (only for image uploads), Conversion Reviewer (only for sales/conversion goals). Each reviewer has a production-quality system prompt tuned to its domain.

PROVIDER ARCHITECTURE:
9 provider adapters via adapter pattern — Anthropic, OpenAI, Gemini, OpenRouter, Groq, Z.AI, Ollama Local, Ollama Remote, Llama.cpp. All adapters with image-capable models pass base64 images correctly. Provider Management modal with cards grid, API key inputs, model dropdowns, base URL, temperature/maxTokens/topP/timeout/maxRetries, Save/Test Connection/Remove Key/Clear All buttons, status badges. All keys in localStorage only.

DASHBOARD (16 SECTIONS):
All rendered from live API responses — Upload Preview, AI Reviewer Activity Timeline, Overall Content Intelligence Score (circular SVG ring), Category Score Breakdown (bar chart), AI Reasoning (per-reviewer cards), Strength Analysis, Weakness Analysis, Missed Opportunities, Platform-Specific Recommendations, Improved Content Version (best across reviewers), Alternative Hooks (deduped), Alternative Titles (deduped), Publishing Checklist (toggleable), Generate Final Content (with Use Recommended button + Before vs After comparison), Final Executive Report (Executive Summary, Content Health Report, Highest Impact Improvements, AI Estimated Performance Potential, Before vs After Comparison, Advanced Optimization Prompts).

ERROR HANDLING:
3-attempt exponential backoff with jitter, AbortSignal.timeout, rate-limit detection (429), network failure recovery, toast notifications for every reviewer start/complete/error.

VISUAL DESIGN:
Commercial SaaS quality. Dark mode (#09090B). Glassmorphism. Animated gradients. Professional spacing. Modern typography (Inter + JetBrains Mono). Responsive. Inline SVG icons only. Microinteractions.

CODE QUALITY:
Clean, modular, readable JavaScript organized into reusable classes and functions. No syntax errors. No TODO comments. No placeholders. No broken UI.
```

---

## Features

- **5-question interview configuration** — the app asks one multiple-choice question at a time (content type → platform → goal → upload type → review intensity) in a chat-like interface. Each question includes professional options plus "Other" with free-text fallback. No analysis runs until the interview is complete. A progress indicator and dot navigation show "X/5" completion.
- **Dynamic reviewer selection** — based on the interview answers, the app selects 3–6 specialized reviewers from a pool of 6: Content Strategy Reviewer (always), Platform Growth Reviewer (always), Audience Psychology Reviewer (always), Engagement Optimization Reviewer (always for engagement/education goals), Visual Design Reviewer (only for image uploads), Conversion Reviewer (only for sales/lead-generation goals). Each reviewer has a production-quality system prompt tuned to its domain that instructs the model to emit section-marker headers.
- **6 specialized reviewers with production system prompts** — each reviewer's system prompt is context-aware (it receives the content type, platform, goal, and intensity from the interview) and instructs the model to respond in natural language using exact `## MARKER:` section headers. The reviews are parsed by section markers — no JSON forcing, no strict JSON parsing, no fragile schema enforcement.
- **Real sequential reviewer pipeline** — a genuine `for` loop runs each selected reviewer one at a time. Every reviewer call is a real API request — no mocks, no fabricated scores, no regex-generated reasoning. Each reviewer's output is displayed in the Activity Timeline the moment it completes, and the dashboard updates incrementally.
- **Provider-agnostic REST API integration** — supports 10 providers via a unified adapter pattern:
  - **Z.AI Proxy (Built-in)** — a backend proxy that calls the Next.js `/api/day47-chat` route using `z-ai-web-dev-sdk` server-side. **No API key required.** Works out-of-the-box. This is the default provider.
  - Anthropic (Claude) — `x-api-key` header + `anthropic-dangerous-direct-browser-access`
  - OpenAI — `Bearer` auth
  - Google Gemini — `?key=` query parameter
  - OpenRouter — `Bearer` auth (OpenAI-compatible)
  - Groq — `Bearer` auth (OpenAI-compatible, text-only)
  - Z.AI — `Bearer` auth (OpenAI-compatible)
  - Ollama (Local) — local LLM support
  - Ollama (Colab/Remote) — remote Ollama instance support
  - Llama.cpp (Local) — local llama.cpp server support
  - Adding a new provider requires only implementing a new adapter — the orchestration engine never changes.
- **Backend proxy route** — the `/api/day47-chat` Next.js API route uses `z-ai-web-dev-sdk` server-side, reuses a single warmed ZAI instance, supports multimodal (image_url) content, defaults to `glm-4.6`, has 120s `maxDuration`, and mirrors the rateLimited flag for the client's retry logic. This means the studio works instantly on first load with zero configuration.
- **File upload support (.txt, .md, .docx)** — in addition to pasting text, the user can click "Upload Text File" to load content from a `.txt`, `.md`, or `.docx` file. Plain-text formats (`.txt`, `.md`) are read directly via `file.text()`. For `.docx`, the handler reads the file as an ArrayBuffer, decompresses the zip entries, locates `word/document.xml`, and extracts text from `<w:t>` elements — so Word documents load as readable text without any external library.
- **Provider management modal** — provider cards with password-type API key inputs (show/hide toggle), model selection (dropdown for cloud providers, free text for local), base URL, temperature, max tokens, top-P, timeout, max retries. Active provider indicator. Status badges (Not Configured, Ready, Connected, Error). Save, Test Connection, Remove Key, Clear All actions. All keys stored in localStorage only. Switch providers without reloading.
- **16-section dashboard** — all rendered from live API responses:
  1. **Upload Preview** — the original content with word/char count + New Analysis button
  2. **AI Reviewer Activity Timeline** — real-time pending → running → complete → error status for each reviewer, with per-reviewer scores
  3. **Overall Content Intelligence Score** — circular SVG ring with gradient, animated score counter, letter grade (Weak/Fair/Good/Strong/Elite), one-line headline verdict
  4. **Category Score Breakdown** — animated horizontal bar chart showing each reviewer's score (0–100) with color-coded bars
  5. **AI Reasoning** — per-reviewer cards showing each reviewer's full reasoning paragraph
  6. **Strength Analysis** — deduped list of strengths across all reviewers, with count badge
  7. **Weakness Analysis** — deduped list of weaknesses across all reviewers, with count badge
  8. **Missed Opportunities** — deduped list of opportunities the content is missing
  9. **Platform-Specific Recommendations** — tagged by platform (LinkedIn, Twitter, etc.)
  10. **Improved Content Version** — the best improved version across all reviewers, with source attribution
  11. **Alternative Hooks** — 3–5+ deduped alternative opening hooks
  12. **Alternative Titles** — 3–5+ deduped alternative titles/headlines
  13. **Publishing Checklist** — actionable, toggleable checklist derived from weaknesses + opportunities
  14. **Generate Final Content** — a "Use Recommended (Hook #1 + Title #1)" button plus a "Generate Final Content" button that produces a publish-ready version integrating the improved draft, selected hook, and selected title
  15. **Before vs After Comparison** — side-by-side comparison with a red "BEFORE" badge (original draft) and green "AFTER" badge (AI-polished version), separated by a purple arrow. Copy Final Content button.
  16. **Final Executive Report** — one extra API call that synthesizes all reviewer outputs into: Executive Summary, Content Health Report, Highest Impact Improvements, AI Estimated Performance Potential, Before vs After Comparison, Advanced Optimization Prompts.
- **Natural-language section-marker parsing** — both the reviewer response parser and the executive report parser use word-boundary regex matching with an optional colon (`## MARKER` or `## MARKER:`). This is robust to minor prompt-compliance variation — the model sometimes emits bare headers and sometimes adds a colon, and both parse correctly.
- **Error handling** — retries with exponential backoff + jitter on 429/5xx/network errors. Timeout handling via `AbortSignal.timeout`. Rate-limit detection. Recovery-suggestion toasts. Per-reviewer error isolation — one failed reviewer doesn't kill the whole run.
- **Visual design** — dark mode (#09090B background), glassmorphism panels, animated gradient dividers, conic-gradient brand mark, pulse/glow/shine animations, Inter + JetBrains Mono fonts, inline SVG icons only, loading spinners, hover effects, toast notifications, responsive layout (2-column dashboard grid collapses to 1 column on mobile).

---

## Screenshots

### Interview — Question 1 (Content Type)
![Interview Q1](Screenshots/01-interview-q1.png)

### Interview — Question 2 (Platform)
![Interview Q2](Screenshots/02-interview-q2.png)

### Interview — Question 3 (Goal)
![Interview Q3](Screenshots/03-interview-q3.png)

### Interview — Question 4 (Upload Type)
![Interview Q4](Screenshots/04-interview-q4.png)

### Interview — Question 5 (Review Intensity)
![Interview Q5](Screenshots/05-interview-q5.png)

### Interview Complete
![Interview Complete](Screenshots/06-interview-complete.png)

### Upload Studio
![Upload Studio](Screenshots/07-upload-studio.png)

### Analysis Started
![Analysis Started](Screenshots/08-analysis-started.png)

### Analysis Running (Activity Timeline)
![Analysis Running](Screenshots/09-analysis-running.png)

### Dashboard Overview
![Dashboard Overview](Screenshots/10-dashboard-overview.png)

### Overall Content Intelligence Score
![Overall Score](Screenshots/11-overall-score.png)

### Category Score Breakdown
![Score Breakdown](Screenshots/12-score-breakdown.png)

### AI Reasoning (Per-Reviewer Cards)
![AI Reasoning](Screenshots/13-ai-reasoning.png)

### Strength Analysis
![Strengths](Screenshots/14-strengths.png)

### Weakness Analysis
![Weaknesses](Screenshots/15-weaknesses.png)

### Improved Content Version
![Improved Version](Screenshots/16-improved-version.png)

### Alternative Hooks
![Hooks](Screenshots/17-hooks.png)

### Alternative Titles
![Titles](Screenshots/18-titles.png)

### Generate Final Content Panel
![Generate Panel](Screenshots/19-generate-panel.png)

### Use Recommended Selected
![Recommended Selected](Screenshots/20-recommended-selected.png)

### Before vs After Comparison
![Before vs After](Screenshots/21-before-after.png)

### Final Executive Report
![Executive Report](Screenshots/22-executive-report.png)

---

## Technologies Used

- HTML5
- CSS3 (glassmorphism, animated gradients, grid/flex layouts, custom scrollbars, responsive media queries)
- Vanilla JavaScript (ES6+ classes, async/await, fetch API, state management, section-marker parsing)
- SVG (circular score ring, bar chart, inline icons)
- Google Fonts (Inter, JetBrains Mono)
- Next.js 16 API Route (the `/api/day47-chat` backend proxy)
- TypeScript 5 (the backend proxy route)
- `z-ai-web-dev-sdk` (server-side LLM calls via the Z.AI GLM-4.6 model)
- No external frontend libraries, frameworks, or CDNs

---

## Key Learnings

### Technical Learnings

- **A backend proxy is the right default for demo-able AI apps.** The original design called external LLM APIs directly from the browser, which meant every user had to bring their own API key before seeing a single result. Adding a built-in `Z.AI Proxy` adapter backed by a Next.js API route using `z-ai-web-dev-sdk` meant the studio works instantly on first load — zero configuration, zero key juggling. The adapter pattern made this a drop-in addition: one new class, one new entry in the `PROVIDERS` array, zero changes to the orchestration engine.
- **Natural-language section markers beat JSON forcing for long-form AI output.** Early versions tried to make reviewers return JSON. Models complied inconsistently — they'd wrap JSON in markdown fences, truncate long fields, or escape quotes wrong. Switching to `## MARKER:` section headers in natural language meant the model writes freely (which it does well) and the parser extracts sections by position. The fix that made this fully robust was making the colon **optional** with word-boundary matching — the executive report prompt emits bare `## EXECUTIVE SUMMARY` headers while reviewers emit `## SCORE: 85/100`, and one parser handles both.
- **The `STATE.uploadedText` vs `STATE.textContent` bug was a silent failure.** The Before vs After panel referenced `STATE.uploadedText` — a field that was never assigned anywhere in the codebase. The real uploaded text lives in `STATE.textContent`. Because JavaScript returns `undefined` for missing properties and `||` falls through, the panel silently showed the `(image upload)` placeholder for every text upload. The lesson: a typo in a state field name produces no error, no warning — just wrong output. A simple lint rule or a state-shape check would have caught it.
- **Side-by-side panels in a 2-column grid produce identical screenshots.** When two panels (strengths/weaknesses, hooks/titles) sit in the same grid row, a full-viewport screenshot captures both at once — so "scroll to panel A" and "scroll to panel B" produce the same image. The fix was to offset the second capture by a few hundred pixels so the viewport framing differs.
- **Word documents are zip files, not XML files.** The original `.docx` handler read the file as raw bytes and grepped for `<w:t>` tags — which never matches because `word/document.xml` is deflate-compressed inside the zip. The fix reads the file as an ArrayBuffer, walks the zip central directory, inflates the `word/document.xml` entry, and then extracts text from `<w:t>` elements. All client-side, no library.

### Conceptual Learnings

- **Multiple specialized reviewers beat one generalist reviewer.** A single "review this content" prompt produces a single generic score and a few bullet points. Six specialized reviewers — each with a domain-tuned system prompt — produce six different perspectives that, when synthesized, surface insights no single reviewer would have found. The Content Strategy Reviewer sees structure; the Audience Psychology Reviewer sees emotional resonance; the Conversion Reviewer sees the call-to-action. The executive report's job is to reconcile them.
- **Dynamic reviewer selection respects the user's intent.** Deploying the Visual Design Reviewer for a text-only upload wastes an API call. Deploying the Conversion Reviewer for a thought-leadership goal misses the point. Tying reviewer selection to the interview answers means every run deploys exactly the reviewers that matter for that specific content — no more, no less.
- **The Before vs After comparison is the moment of truth.** Every other dashboard section is analysis. The Before vs After is the payoff — the user sees their original draft next to the AI-polished version and immediately judges whether the whole pipeline was worth it. Making this distinction visually unmistakable (red BEFORE badge + green AFTER badge + center arrow) isn't decoration — it's the difference between "I see two text boxes" and "I see the transformation."

### Personal Reflection

The hardest bug in this build was invisible. The Before vs After panel showed "(image upload)" for every text upload — not because the analysis failed, not because the API errored, but because one line of code referenced `STATE.uploadedText` instead of `STATE.textContent`. The screenshots looked fine on casual inspection (the After column had content), so the bug shipped. It took a vision model staring at the screenshot and reporting "the BEFORE column shows (image upload)" to surface it. The lesson: when a panel has two columns and one is a placeholder, the eye fills in the gap. You need to actively verify each side, not just confirm "there's content." Fixing it — pointing the code at the real state field, adding explicit BEFORE/AFTER badges, dropping a purple arrow between the columns — turned the weakest screenshot into one of the clearest. The layout evolution taught me that "showing a comparison" isn't enough; the comparison has to *look* like a comparison, or the user reads it as two unrelated text boxes.

---

## Project Structure

```
Day47/
├── index.html
├── day47.md
└── Screenshots/
    ├── 01-interview-q1.png
    ├── 02-interview-q2.png
    ├── 03-interview-q3.png
    ├── 04-interview-q4.png
    ├── 05-interview-q5.png
    ├── 06-interview-complete.png
    ├── 07-upload-studio.png
    ├── 08-analysis-started.png
    ├── 09-analysis-running.png
    ├── 10-dashboard-overview.png
    ├── 11-overall-score.png
    ├── 12-score-breakdown.png
    ├── 13-ai-reasoning.png
    ├── 14-strengths.png
    ├── 15-weaknesses.png
    ├── 16-improved-version.png
    ├── 17-hooks.png
    ├── 18-titles.png
    ├── 19-generate-panel.png
    ├── 20-recommended-selected.png
    ├── 21-before-after.png
    └── 22-executive-report.png

src/app/api/day47-chat/
└── route.ts                      (backend proxy using z-ai-web-dev-sdk)

public/
├── day47.html                    (served at /day47.html)
└── sample-content.md             (sample article for upload testing)
```

---

## Final Thoughts

This project is a study in building a genuine multi-reviewer AI content analysis pipeline — not a single-prompt wrapper. Six specialized reviewers each reason independently with domain-tuned system prompts. Every score, strength, weakness, and improved version originates from a live API call. The reviews are parsed by natural-language section markers, not fragile JSON. A final executive report synthesizes all reviewer outputs into one executive-ready brief. The provider architecture is production-grade — 10 providers via an adapter pattern, including a built-in backend proxy that works instantly with no API key. File uploads support `.txt`, `.md`, and `.docx` (with real zip decompression for Word documents). The Before vs After comparison — the moment of truth — uses explicit red/green badges and a center arrow so the transformation is unmistakable. Open the HTML file in any browser, walk the 5-question interview, paste or upload your content, and watch 4–6 AI reviewers dissect it from different angles before a final synthesis tells you exactly what to fix and how. That's what a content consultant does.
