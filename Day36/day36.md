# Day 36 – Cognitive Pattern Explorer

## Overview

A single-file HTML/CSS/JavaScript application that maps how you think across everyday scenarios, priority rankings, and decision sequences — then compares your patterns under calm versus stress conditions.

The core problem it addresses is one that most self-reflection tools get wrong: they ask you to describe yourself in the abstract, which rarely matches how you actually decide things in the moment. This app instead puts you through 10 concrete scenarios ("Your manager offers you a new role with more visibility but an unclear scope"), forces you to pick one of four responses, and infers your thinking style from the pattern of those choices. It then layers on a priority builder and a thinking timeline to cross-reference what you say you value against how you actually sequence a decision.

The educational objective is self-reflection, not assessment. The app says this explicitly in a disclaimer box, and it's worth taking seriously — the results shift depending on which scenarios get sampled and how honest you are with yourself.

---

## Features

- **20+ scenario pool** across career, ethics, leadership, money, time pressure, communication, and conflict resolution. Ten are sampled per session, shuffled both in order and in answer position.
- **Two modes**: Calm (no timer, soft pacing) and Stress (countdown timer on every scenario, urgency in the wording). The same scenarios play differently depending on which mode you're in.
- **Priority Builder**: drag-to-reorder 10 values (Accuracy, Speed, Learning, Security, Creativity, Recognition, Collaboration, Curiosity, Challenge, Stability) from most to least important.
- **Thinking Timeline**: drag-to-reorder 6 decision steps (Observe, Research, Reflect, Discuss, Act, Review) to reflect your natural sequence.
- **Reflection Journal**: generates a radar chart, distribution bars, a cross-chapter summary, personalized observations, strengths/blind spots, and practical habits based on your dominant thinking style.
- **Calm vs Stress comparison**: unlocked once you complete both modes, showing side-by-side distribution shifts and a generated insight about how your thinking changed.
- **XP, achievements, streaks, and sound effects** — light gamification that doesn't get in the way.
- **Export to TXT/JSON, print support, and session persistence** via localStorage.

The three fixes I made to the original file before running through it:

1. **Stress mode scenario timer was static.** The countdown showed a number but never changed. The original code used `setInterval` with `remaining--` decremented once per tick, but the interval wasn't firing reliably and the display stayed stuck. Replaced it with a `Date.now()`-based elapsed-time calculation that updates every 200ms — the bar and text now count down smoothly.
2. **No selection state on answer choices.** Hover gave a color change, but once you clicked there was no persistent visual indicator of which option you picked. Added a `.selected` class with a filled background, accent border, glow, and a `::before` checkmark, plus a `.dimmed` class that fades the non-selected options.
3. **No countdown on Priority Builder and Thinking Timeline in stress mode.** These screens gave you unlimited time even in stress mode, which broke the pressure dynamic. Added a 30/45/60-second selector that appears only in stress mode — you pick a duration, the selector hides, and a countdown bar starts. When it hits zero, the list locks.

---

## Screenshots

### Start Screen (Calm Mode Selected)

![Calm mode home screen](Screenshots/calm-01-home.png)

The mode select screen. Calm mode is selected — you can see the accent border and tinted background on the Calm card. The "Begin Chapter 1" button is enabled.

### Chapter 1 — Calm Scenarios

Three representative scenarios from the calm mode run. Each shows the scenario text, four shuffled choices, the selected answer (highlighted with checkmark and accent border), the dimmed non-selected options, and the "why" explanation box that appears after selecting.

![Calm scenario 1 — Career](Screenshots/calm-01-scenario.png)

![Calm scenario 5 — Technology](Screenshots/calm-05-scenario.png)

![Calm scenario 8 — Ethics](Screenshots/calm-08-scenario.png)

### Chapter 2 — Priority Builder (Calm)

Default order (left) versus my reordered list (right). The default is alphabetical; my calm-mode order puts Accuracy, Learning, and Security at the top — reflecting a curiosity-driven, evidence-before-action priority set.

![Priority Builder — default order](Screenshots/calm-11-priority-default.png)

![Priority Builder — my calm order](Screenshots/calm-12-priority-mine.png)

### Chapter 3 — Thinking Timeline (Calm)

Default shuffled order versus my natural sequence. In calm conditions, I start with Observe, move to Research, then Reflect, Discuss, Act, and finally Review — a fairly linear analytical flow.

![Thinking Timeline — default order](Screenshots/calm-13-timeline-default.png)

![Thinking Timeline — my calm order](Screenshots/calm-14-timeline-mine.png)

### Reflection Journal (Calm)

The full report after completing calm mode. Shows 70% Analytical Thinker, 85/100 reflection score, the radar chart, and the "Do Stress Mode" prompt since comparison isn't unlocked yet.

📄 **Calm Reflection Journal:** [calm-journal-report.pdf](journals/calm-journal-report.pdf)

### Chapter 1 — Stress Scenarios

Same scenario structure, but now with a visible countdown timer at the top. The timer bar depletes and the text shows remaining seconds. The selected state and dimming work identically to calm mode.

![Stress scenario 2 — College](Screenshots/stress-02-scenario.png)

![Stress scenario 5 — AI](Screenshots/stress-05-scenario.png)

![Stress scenario 9 — Time Pressure](Screenshots/stress-09-scenario.png)

### Chapter 2 — Priority Builder (Stress)

The stress version adds a 30/45/60-second timer selector before you can start reordering. Default order shown first, then my stress-mode order — note how Security jumps to #1 and Stability moves up to #3. The shift from "optimize for growth" to "optimize for protection" is visible in the ranking.

![Priority Builder — stress default with timer selector](Screenshots/stress-11-priority-default.png)

![Priority Builder — my stress order with timer running](Screenshots/stress-12-priority-mine.png)

### Chapter 3 — Thinking Timeline (Stress)

Same timer selector pattern. My stress timeline puts Research first (scanning for risks before observing) and keeps Review at the end — the sequence becomes more vigilant and verification-focused.

![Thinking Timeline — stress default with timer selector](Screenshots/stress-13-timeline-default.png)

![Thinking Timeline — my stress order with timer running](Screenshots/stress-14-timeline-mine.png)

### Reflection Journal (Stress, with Comparison)

The stress journal shows 50% Analytical, 30% Overthinking, 20% Action — and crucially, the Calm vs Stress comparison is now unlocked. The side-by-side distribution and the generated observation ("pressure intensified the tendency to second-guess rather than speed up") capture the actual shift better than I expected.

📄 **Stress Reflection Journal:** [calm-journal-report.pdf](journals/calm-journal-report.pdf)

### Calm vs Stress Comparison (Close-up)

A dedicated view of the comparison section. The side-by-side percentages make the shift visible at a glance: Analytical drops from 70% to 50%, Overthinking rises from 0% to 30%, and Action edges up from 10% to 20%. The "Interesting Observation" bullets quantify the shift — decisions were 21% slower under stress (more second-guessing, not more speed), and choices leaned more action-oriented despite the overthinking increase.

📄 **Comparison Report:** [comparison-report.pdf](journals/comparison-report.pdf)
---

## Technologies Used

- HTML5
- CSS3 (glassmorphism, backdrop-filter, CSS variables, drag-and-drop styling)
- Vanilla JavaScript (no dependencies, no frameworks)
- Canvas API (radar chart, confetti)
- WebAudio API (offline sound effects)
- localStorage (session persistence and history)

---

## Key Learnings

### Technical Learnings

- **Timer accuracy matters more than interval frequency.** The original stress timer used `setInterval` with a counter variable decremented once per tick. If the browser throttled the interval (which it does for background tabs), the countdown drifted. Replacing it with `Date.now()`-based elapsed-time math means the displayed seconds always reflect real elapsed time, regardless of throttling. The 200ms update interval is just for visual smoothness — the underlying calculation is time-based, not tick-based.
- **CSS `::before` is cleaner than extra DOM nodes for selection indicators.** Adding a checkmark via `content: "✓ "` in a pseudo-element avoids injecting extra spans or icons into the DOM, which keeps the markup clean and the `textContent` matching logic (used for identifying choices) unaffected.
- **User-selectable countdowns need a setup-then-run pattern.** The timer selector has to hide itself after selection, then reveal the countdown bar. Using a `.hidden` class toggle on two sibling elements is simpler than conditionally rendering HTML, and it preserves the ability to inspect the selector's state in DevTools.
- **`dataset.picked` is a lightweight lock.** Using a data attribute to mark buttons as "already answered" prevents double-clicks without needing a separate boolean variable in closure scope. It also survives re-renders of the parent container, which a closure variable wouldn't.

### Conceptual Learnings

- **The calm-vs-stress contrast isn't "logical vs. fast."** The default framing in many decision-style tools is that stress makes you faster and more intuitive. My actual data showed the opposite: under stress, my analytical percentage dropped from 70% to 50%, but overthinking rose from 0% to 30%. The shift was toward more self-critical analysis, not toward action. The app's comparison insight — "pressure intensified the tendency to second-guess rather than speed up" — captured this accurately once the answers reflected it.
- **Priority reordering is more honest than rating scales.** When you rate "how important is Security?" on a 1-5 scale, everything tends to cluster at 4. Forced ranking prevents that. Seeing Security jump from #3 (calm) to #1 (stress) and Stability from #5 to #3 is a clearer signal than any Likert data would give.
- **The scoring algorithm rewards variety, not consistency.** The reflection score factors in how many different thinking styles you drew on. This is a design choice — it frames cognitive flexibility as a strength. Whether that's the right framing depends on what you're optimizing for. A specialist who consistently applies one style might score lower but be more effective in their domain.

### Personal Reflection

The most useful thing the app surfaced wasn't a surprise — I already knew I lean analytical and that stress makes me more risk-aware. What was useful was seeing it quantified side by side. The 20-point drop in analytical and the 30-point rise in overthinking under stress is a concrete shift, not a vague feeling. The priority reorder told the same story from a different angle: Security and Stability moving up under stress isn't indecision, it's a shift from "what could I gain?" to "what could I lose?" That distinction — growth-optimization versus protection-optimization — is something I'll notice in real decisions now that I've seen it in the data.

---

## Project Structure

```
day36/
├── cognitive-pattern-explorer.html
├── day36.md
├── Screenshots/
│   ├── calm-01-home.png
│   ├── calm-01-scenario.png
│   ├── calm-02-scenario.png
│   ├── calm-03-scenario.png
│   ├── calm-04-scenario.png
│   ├── calm-05-scenario.png
│   ├── calm-06-scenario.png
│   ├── calm-07-scenario.png
│   ├── calm-08-scenario.png
│   ├── calm-09-scenario.png
│   ├── calm-10-scenario.png
│   ├── calm-11-priority-default.png
│   ├── calm-12-priority-mine.png
│   ├── calm-13-timeline-default.png
│   ├── calm-14-timeline-mine.png
│   ├── stress-01-scenario.png
│   ├── stress-02-scenario.png
│   ├── stress-03-scenario.png
│   ├── stress-04-scenario.png
│   ├── stress-05-scenario.png
│   ├── stress-06-scenario.png
│   ├── stress-07-scenario.png
│   ├── stress-08-scenario.png
│   ├── stress-09-scenario.png
│   ├── stress-10-scenario.png
│   ├── stress-11-priority-default.png
│   ├── stress-12-priority-mine.png
│   ├── stress-13-timeline-default.png
│   └── stress-14-timeline-mine.png
│
└── journals/
    ├── calm-journal-report.pdf
    ├── stress-journal-report.pdf
    └── comparison-report.pdf
```

---

## Final Thoughts

The app does what it set out to do: it surfaces decision patterns through concrete scenarios rather than abstract self-reporting, and the calm/stress comparison adds a dimension that single-pass tools miss. The three fixes I made (timer countdown, selection state, stress-mode timers on the builder screens) were all about making the stress condition actually feel stressful — without them, "stress mode" was just calm mode with orange text. With them, the countdown creates enough pressure to shift behavior, which is the whole point.

The scoring won't fit everyone. It rewards variety and flags overthinking as a blind spot, which is a defensible design choice but not the only one. If the report consistently feels off after honest answers, that's a signal to adjust the scoring weights, not to game the inputs.
