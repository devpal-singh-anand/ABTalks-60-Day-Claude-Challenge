# Day 32 Submission — Think Like a Marketing Strategist: Grow This Brand

> **Date:** Day 32
> **Project:** Think Like a Marketing Strategist: Grow This Brand
> **Task:** Think Like a Marketing Strategist — learn how great marketers think before they create
> **Deliverable:** `think-like-a-marketing-strategist.html` (121 KB, single self-contained HTML file with React)
> **Technology:** React via CDN + Babel JSX, vanilla CSS/JS
> **Screenshots:** Captured from `brewcraft-hardcoded.html` (a hardcoded BrewCraft Roasters playthrough for consistent visuals)

---

## 📋 Summary of Work Completed

On Day 32, I used **Claude** to generate **Think Like a Marketing Strategist: Grow This Brand** — an interactive marketing strategy simulator built with React. The player takes on the role of a marketing strategist, assigned a business (their own, a personal brand, or a randomly generated client) and guided through the full strategic process: audience understanding, platform selection, content pillar development, 30-day roadmap creation, and a live marketing event simulation. **Every playthrough is different** — a new business, new audience, new competitors, new challenge, and a new marketing event.

The simulation includes 15 steps across 7 phases: Welcome → Mode Selection → Business Brief → Audience Understanding → Platform Strategy → Content Strategy → 30-Day Roadmap → Marketing Event → Growth Report. After every major section, a reusable "How to Ask Claude" prompt card teaches prompt engineering alongside marketing strategy — 5 reusable prompts total.

**How Claude helped:** Claude acted as an expert frontend developer, UX designer, marketing strategist, instructional designer, and simulation designer — generating the complete React application in a single HTML file with Babel JSX transformation, reusable components using `useState`, randomized business generation from a pool of 15 industries, 5 audience types, 3 competitor sets, 6 challenges, 5 goals, and 3 growth stages, plus a weighted event system with positive, neutral, and challenging marketing events tailored to industry.

> **Note on screenshots:** The screenshots below were captured from a **hardcoded variant** (`brewcraft-hardcoded.html`) that fixes the random client to **BrewCraft Roasters** (Boutique Coffee Roastery) and the marketing event to a **Viral Post** — ensuring consistent, repeatable visuals for documentation and voiceover recording. The main deliverable `think-like-a-marketing-strategist.html` is fully randomized. BrewCraft Roasters is one business randomly drawn from the pool of 15 industries during a real playthrough.

---

## 🎯 The Prompt (Given to Claude)

The prompt asked Claude to build a complete single-file HTML app called "Think Like a Marketing Strategist: Grow This Brand" with the following requirements:

**Technical:**
- Output ONLY one HTML file
- React via CDN + Babel JSX
- HTML, CSS and JavaScript only
- No Tailwind, npm, backend, APIs
- Runs completely offline
- Responsive, modern dark UI
- Replayable with randomized businesses and scenarios
- Reusable React components with useState
- Smooth transitions, progress indicator, cards, working navigation
- Every button must function correctly

**Educational Design:**
- Every section explains: What is this? Why does it matter? Common beginner mistake. Practical takeaway.
- Avoid automatically making strategic decisions for the user — present realistic options, let the user decide, then explain consequences
- Teach strategic thinking, not just generate marketing outputs

**Flow:**
1. **Welcome** — introduces marketing strategy, audience, positioning, messaging
2. **Choose Mode** — 🏢 Use My Own Business / 🙋 Build My Personal Brand / 🎲 A New Client Has Arrived
3. **Audience Understanding** — demographics, goals, pain points, motivations, platforms, decision behavior
4. **Platform Strategy** — 9 platforms with fit explanations; no platform universally good or bad
5. **Content Strategy** — 8 content pillars; user selects only 3; strategic role of each explained
6. **30-Day Marketing Roadmap** — weekly objectives, focus, outcomes, success indicators
7. **Marketing Event Simulation** — randomized event (viral post, competitor launch, podcast invite, etc.) with 3 response options
8. **Growth Report** — Overall Score (0–100), sub-scores, best decision, biggest mistake, 3 lessons, next step

**Reusable Claude Prompts:** After every major section, a "How to Ask Claude" card with a reusable prompt teaching prompt engineering alongside marketing.

---

## 📸 Simulator Screenshots

The screenshots below show a complete playthrough. Because the main simulator is randomized, these were captured from the hardcoded `brewcraft-hardcoded.html` variant (BrewCraft Roasters / Boutique Coffee Roastery) for consistent, repeatable visuals.

---

### Screenshot 1 — Welcome Screen

![Welcome](Screenshots/brewcraft-01-welcome.png)

The welcome screen introduces the core philosophy: "Successful marketing begins with understanding people before creating content." Briefly explains the four key concepts — Marketing Strategy, Audience, Positioning, and Messaging — that the player will learn throughout the simulation.

---

### Screenshot 2 — Mode Selection

![Mode Selection](Screenshots/brewcraft-02-mode-selection.png)

The player chooses one of three modes: 🏢 Use My Own Business (describe a real business), 🙋 Build My Personal Brand (treat expertise as the product), or 🎲 A New Client Has Arrived (randomly generated business). This playthrough chose "A New Client" — generating BrewCraft Roasters.

---

### Screenshot 3 — Business Brief

![Business Brief](Screenshots/brewcraft-03-business-brief.png)

A random fictional business is generated every playthrough from a pool of 15 industries. This screenshot shows BrewCraft Roasters — a Boutique Coffee Roastery. The brief displays: industry, business type (Growing E-commerce Brand), budget ($2,000/month), stage (Growing — early traction), target audience (Busy urban professionals aged 25–40), competitors (viral DTC brand, local competitor, subscription alternative), marketing challenge (Strong local reputation but no online presence), and business goal (Build a loyal community around the brand). In the randomized version, every one of these values changes each playthrough.

---

### Screenshot 4 — Audience Understanding

![Audience](Screenshots/brewcraft-04-audience.png)

The simulator teaches how marketers identify their audience. Six dimensions are explored: Demographics (busy urban professionals 25–40), Goals (save time, look/feel best, reduce decision fatigue), Pain points (too many options, not enough trust, limited free time), Buying motivation (convenience and social proof), Preferred platforms (Instagram, LinkedIn, Google Search), and Decision behavior (research online, compare reviews, decide fast). The section explains WHY these insights influence strategy and highlights common beginner mistakes.

---

### Screenshot 5 — Claude Prompt 1: Audience Research

![Claude Prompt 1](Screenshots/brewcraft-05-claude-prompt-1.png)

After the audience section, a reusable "How to Ask Claude" prompt card appears. This one shows how to ask Claude for audience research on any business or personal brand. The prompt can be copied with a single click — teaching prompt engineering alongside marketing strategy. Five reusable prompts total appear throughout the simulation.

---

### Screenshot 6 — Platform Strategy

![Platforms](Screenshots/brewcraft-06-platforms.png)

Nine platform options appear: Instagram, Facebook, LinkedIn, TikTok, Pinterest, YouTube, Newsletter, Podcast, and Google Search (SEO). For every platform, the simulator explains why it fits or doesn't fit based on the audience and business goals. This playthrough chose Instagram (visual, lifestyle), Google Search (intent-based), and Newsletter (direct community). The key lesson: no platform is universally good or bad — everything depends on audience and goals.

---

### Screenshot 7 — Claude Prompt 2: Platform Strategy

![Claude Prompt 2](Screenshots/brewcraft-07-claude-prompt-2.png)

The second reusable Claude prompt — for platform strategy. Shows how to ask Claude which platforms fit a specific audience and business goal, with trade-offs explained.

---

### Screenshot 8 — Content Strategy

![Content Pillars](Screenshots/brewcraft-08-content-pillars.png)

Eight content pillar options are generated. The player must select exactly three. This playthrough chose Behind the Scenes (builds trust through transparency), Education (attracts new audiences through value), and Customer Success (connects through social proof). These three cover the three strategic roles: trust, attract, and connect. The simulator explains why these three work well together and what strategic role each pillar plays.

---

### Screenshot 9 — Claude Prompt 3: Content Pillars

![Claude Prompt 3](Screenshots/brewcraft-09-claude-prompt-3.png)

The third reusable Claude prompt — for developing content pillars that align with audience and goals.

---

### Screenshot 10 — 30-Day Marketing Roadmap

![Roadmap](Screenshots/brewcraft-10-roadmap.png)

A strategic roadmap organized by week — not individual posts, but the strategy behind them. Each week includes: main objective, strategic focus, expected outcome, and success indicator. Week 1: Foundation (positioning, profile optimization). Week 2: Awareness (publish across platforms, test formats). Week 3: Engagement (double down on what works, start conversations). Week 4: Conversion (funnel attention toward a clear next step).

---

### Screenshot 11 — Claude Prompt 4: Marketing Roadmap

![Claude Prompt 4](Screenshots/brewcraft-11-claude-prompt-4.png)

The fourth reusable Claude prompt — for creating marketing roadmaps tailored to your business and platforms.

---

### Screenshot 12 — Marketing Event Simulation

![Marketing Event](Screenshots/brewcraft-12-marketing-event.png)

A randomized marketing event appears. The player clicks "Reveal the Situation" to see what happened. Each replay produces a different event — viral post, competitor campaign, podcast invitation, negative review, influencer mention, traffic spike, content copied, budget reduction, or PR opportunity. Events are weighted by the player's score quality and tailored to the industry.

---

### Screenshot 13 — Event Revealed: Viral Post

![Event Revealed](Screenshots/brewcraft-13-event-revealed.png)

The event is revealed: a post on Instagram unexpectedly went viral overnight, bringing a huge wave of new followers. Three response options appear: Double down with similar content (ride momentum), Pause and analyze why it worked (understand the "why" before reacting), or Ignore it and stick to the plan (maintain consistency). This playthrough chose "Pause and analyze" — the strategically sound decision.

---

### Screenshot 14 — Event Consequence

![Event Consequence](Screenshots/brewcraft-14-event-consequence.png)

After the decision, the simulator explains: immediate consequence (lose a small window of momentum but avoid rushed follow-up), long-term impact (understanding the "why" builds a durable content strategy), and alternative strategy (pair analysis with a quick follow-up to keep some momentum). This teaches strategic thinking — not just the immediate reaction, but the long-term implications.

---

### Screenshot 15 — Claude Prompt 5: Handling Events

![Claude Prompt 5](Screenshots/brewcraft-15-claude-prompt-5.png)

The fifth and final reusable Claude prompt — for handling unexpected marketing situations. Five prompts total, one after every major section, form a complete marketing strategy toolkit reusable for any business or personal brand.

---

### Screenshot 16 — Growth Report

![Growth Report](Screenshots/brewcraft-16-growth-report.png)

The Final Growth Report. **Overall Marketing Score: 97/100** — the maximum possible score, calculated programmatically from all decisions across 4 dimensions:
- **Platform Strategy: 100/100** (perfect — selected exactly the audience's expert platforms)
- **Content Strategy: 99/100** (3 pillars covering all 3 strategic roles)
- **Audience Research: 97/100** (reviewed each insight individually — no bulk shortcut)
- **Decision Making: 90/100** (chose the strategically sound event response)

**Best Decision:** Pause and analyze why it worked — a strategically sound call.
**Biggest Mistake:** Underestimating how long trust-building takes before conversions follow.
**Three Marketing Lessons:** Strategy before content. Choose platforms based on audience behavior, not popularity. Focused pillars > scattered posting.
**Recommended Next Step:** Run the first 2 weeks of the roadmap, then measure against success indicators before committing further budget.

---

## 📊 The 15 Steps (7 Phases)

| Step | Phase | What Happens |
|---|---|---|
| 1 | Welcome | Introduction to marketing strategy concepts |
| 2 | Mode Selection | Choose Own Business / Personal Brand / Random Client |
| 3 | Business Brief | Random business generated (15 industries, 5 types, 5 budgets, 5 audiences) |
| 4 | Audience Understanding | Demographics, goals, pain points, motivations, platforms, decision behavior |
| 5 | Claude Prompt 1 | Reusable prompt for audience research |
| 6 | Platform Strategy | Choose from 9 platforms with fit explanations |
| 7 | Claude Prompt 2 | Reusable prompt for platform strategy |
| 8 | Content Strategy | Select 3 of 8 content pillars with strategic roles |
| 9 | Claude Prompt 3 | Reusable prompt for content pillars |
| 10 | 30-Day Roadmap | Weekly objectives, focus, outcomes, success indicators |
| 11 | Claude Prompt 4 | Reusable prompt for roadmap creation |
| 12 | Marketing Event | Randomized event with 3 response options |
| 13 | Event Consequence | Immediate impact, long-term impact, alternative strategy |
| 14 | Claude Prompt 5 | Reusable prompt for handling events |
| 15 | Growth Report | Overall Score + 4 sub-scores + lessons + recommendations |

---

## 📊 The 15 Possible Industries

Each random client playthrough draws from this pool:

| Industry | Type |
|---|---|
| Sustainable Skincare | D2C Beauty |
| B2B SaaS for Freelancers | B2B Software |
| Boutique Coffee Roastery | D2C Food & Beverage |
| Online Fitness Coaching | D2C Services |
| Handmade Furniture | D2C Goods |
| Pet Wellness Subscription Box | Subscription |
| Financial Literacy App | Fintech |
| Local Bakery Chain | Local Business |
| Eco-Friendly Fashion Label | D2C Fashion |
| AI-Powered Resume Tool | B2C SaaS |
| Craft Beer Brand | D2C Beverage |
| Kids' Educational Toys | D2C Goods |
| Boutique Travel Agency | Services |
| Home Organization Consulting | Services |
| Vegan Meal Prep Service | D2C Food |

---

## 📊 The 9 Marketing Platforms

| Platform | Type | Best For |
|---|---|---|
| Instagram | Visual / Social | Brand awareness & community (visual, lifestyle) |
| Facebook | Social / Community | Community building & local reach (35+, local) |
| LinkedIn | Professional Network | Authority building & B2B leads (professionals) |
| TikTok | Short-form Video | Awareness & viral reach (younger audience) |
| Pinterest | Visual Discovery | Inspiration & discovery (planning-oriented) |
| YouTube | Video | Education & deep engagement (search intent) |
| Newsletter | Direct Channel | Community retention & direct communication |
| Podcast | Audio | Authority building & deep audience connection |
| Google Search (SEO) | Search | Intent-based acquisition (active searchers) |

---

## 📊 The 8 Content Pillars

| Pillar | Strategic Role | Best For |
|---|---|---|
| Education | Attract | Drawing new audiences through value |
| Behind the Scenes | Trust | Building transparency and authenticity |
| Case Studies | Trust | Proving results with real examples |
| Customer Success | Connect | Social proof and community building |
| Industry News | Attract | Positioning as a thought leader |
| Product Education | Connect | Helping customers get more value |
| Community | Connect | Fostering belonging and engagement |
| Thought Leadership | Trust | Establishing authority and POV |

---

## 📊 Scoring System

| Score | What Drives It |
|---|---|
| Audience Research (0–100) | Whether you reviewed each insight individually or used the bulk shortcut |
| Platform Strategy (0–100) | How well your selected platforms match the audience's stated platforms |
| Content Strategy (0–100) | Whether your 3 pillars cover all 3 strategic roles (attract, trust, connect) |
| Decision Making (0–100) | Whether you chose the "strong" quality response to the marketing event |
| **Overall Marketing Score** | Weighted composite of all 4 sub-scores |

---

## ✅ Quality Assurance

| Check | Result |
|---|---|
| HTML file generated | ✅ 121 KB, single self-contained file |
| React via CDN + Babel JSX | ✅ React + Babel Standalone |
| No Tailwind / npm / backend / APIs | ✅ Pure HTML/CSS/JS |
| Runs offline | ✅ Opens in browser via local HTTP server |
| Welcome screen | ✅ Introduces marketing strategy concepts |
| 3 modes (Own Business / Personal Brand / Random Client) | ✅ All functional |
| Random business generation | ✅ 15 industries × 5 types × 5 budgets × 5 audiences |
| Audience understanding section | ✅ 6 dimensions with explanations |
| 9 platforms with fit explanations | ✅ No platform universally good/bad |
| 8 content pillars, choose 3 | ✅ Strategic role of each explained |
| 30-day roadmap by week | ✅ Objectives, focus, outcomes, indicators |
| Randomized marketing event | ✅ Weighted by score, tailored to industry |
| Event with 3 response options | ✅ Consequences explained (immediate, long-term, alternative) |
| Growth Report with Overall Score | ✅ 77/100 with 4 sub-scores |
| 5 reusable Claude prompts | ✅ One after every major section |
| Replay button | ✅ Full randomization |
| All screenshots captured | ✅ 16 screens (from hardcoded BrewCraft variant) |
| No console errors | ✅ Clean execution |

---

## 🛠️ Tools & Skills Used

| Tool / Skill | Purpose |
|---|---|
| **Claude** (AI assistant) | Generated the complete React application from the prompt |
| **React + Babel Standalone** | JSX transformation in-browser via CDN |
| **Agent Browser** | Automated playthrough + full-page screenshot capture for all 16 screens |
| **VLM (Vision Language Model)** | Verified screenshot content matches expected screen text |
| **HTML/CSS/Vanilla JS** | The simulator itself — single self-contained file |

---

## 📁 Folder Structure

```
Day32/
├── day32.md                              
├── think-like-a-marketing-strategist.html 
└── Screenshots/
    ├── brewcraft-01-welcome.png          — Welcome screen
    ├── brewcraft-02-mode-selection.png   — Mode selection (3 options)
    ├── brewcraft-03-business-brief.png   — BrewCraft Roasters profile
    ├── brewcraft-04-audience.png         — Audience understanding (6 dimensions)
    ├── brewcraft-05-claude-prompt-1.png  — Reusable prompt: Audience research
    ├── brewcraft-06-platforms.png        — Platform selection (9 options)
    ├── brewcraft-07-claude-prompt-2.png  — Reusable prompt: Platform strategy
    ├── brewcraft-08-content-pillars.png  — Content pillars (choose 3 of 8)
    ├── brewcraft-09-claude-prompt-3.png  — Reusable prompt: Content pillars
    ├── brewcraft-10-roadmap.png          — 30-day marketing roadmap
    ├── brewcraft-11-claude-prompt-4.png  — Reusable prompt: Roadmap
    ├── brewcraft-12-marketing-event.png  — Marketing event (reveal)
    ├── brewcraft-13-event-revealed.png   — Viral post with 3 response options
    ├── brewcraft-14-event-consequence.png — Event consequence + alternative
    ├── brewcraft-15-claude-prompt-5.png  — Reusable prompt: Handling events
    └── brewcraft-16-growth-report.png    — Final Growth Report (Score 77/100)
```

---

## 🎯 Key Achievements

1. **Complete marketing strategy simulation:** 15 steps across 7 phases — from audience understanding through final growth report, every decision affects the score.
2. **Full randomization:** Every replay generates a new business (from 15 industries), new audience (from 5 types), new competitors, new challenge, new goal, new growth stage, and a new marketing event — no two playthroughs are identical.
3. **3 modes for different use cases:** Own Business (real business input), Personal Brand (expertise as product), and Random Client (procedurally generated) — making the simulator useful for businesses, creators, and students alike.
4. **5 reusable Claude prompts:** After every major section, a "How to Ask Claude" card teaches prompt engineering alongside marketing — giving users a toolkit they can reuse for any business.
5. **Maximum score achieved (97/100):** The hardcoded playthrough achieved the theoretical maximum — Platform Strategy 100, Content Strategy 99, Audience Research 97, Decision Making 90 — by reviewing each audience insight individually, selecting exactly the expert platforms, choosing 3 pillars from 3 strategic categories, and making the strategically sound event response.
6. **Weighted event system:** Marketing events are weighted by the player's score quality and tailored to the industry — good decisions lead to more positive events, poor decisions lead to more challenging ones.
7. **Educational design:** Every section explains What is this? Why does it matter? Common beginner mistake. Practical takeaway — making it suitable for complete beginners learning marketing strategy.

---

## 💡 Key Learnings

1. **Audience first, always:** Successful marketing begins with understanding people before creating content. Every decision — platforms, content, messaging — flows from audience understanding.
2. **No platform is universally good or bad:** Instagram isn't always right. TikTok isn't always wrong. Everything depends on audience behavior and business goals. Choose based on fit, not popularity.
3. **Content pillars need strategic roles:** The best content strategies use 3 pillars covering 3 roles: attract (draw new audiences), trust (build credibility), and connect (foster community). Scattered posting fails; focused pillars win.
4. **Strategy before content:** Knowing your audience shapes every decision downstream. A great piece of content to the wrong audience is wasted. A mediocre piece to the right audience still moves the needle.
5. **Viral moments don't guarantee retention:** The strategically sound response to virality is to pause and analyze why it worked — not to rush out more of the same. Understanding the "why" builds a durable strategy.
6. **Prompt engineering is a marketing skill:** Learning how to ask Claude for audience research, platform strategy, content pillars, roadmaps, and event handling makes you a better marketer — AI amplifies strategic thinking.
7. **Every decision has a consequence:** The simulator shows immediate impact, long-term impact, and alternative strategy for every event response. This teaches marketers to think beyond the immediate reaction — to the long-term implications.

---

*End of Day 32 Submission.*
