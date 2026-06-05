# Day 5 - Context Engineering

Better context. Better outputs.

---

## What I Worked On

Ran the same prompt twice with different levels of context to see how context changes AI output. The prompt asked for a 30-day learning roadmap — simple request, same structure both times.

**Without context:** The prompt had my general situation but Claude still needed more. It asked 3 follow-up questions — what topic I want to learn, my experience level, and how much time I can dedicate daily. I answered those, and only then did Claude generate the roadmap. The result was a broad Career Launchpad covering Python, AI, and Cybersecurity — useful, but generic. Two prompts to get there.

**With context:** The same starting prompt, but Claude had everything it needed. No questions asked. No follow-up needed. It built the roadmap immediately — a focused Data Science & AI Roadmap tailored to my specific skills, goals, and time. One prompt, done.

The key difference isn't just quality. It's that the baseline asked questions. The prompt with context didn't ask for any question and built it right away. Context didn't improve the answer — it eliminated the need for clarification entirely.

I also used the Sider AI browser extension to assist with context management and productivity during the experiment.

---

## The Two Prompts

### Prompt A — Without Context (Baseline)

```
Create a 30-day learning roadmap.

Context:
- Current Situation: Student
- Current Skills: Basic Python and Java, Soft Skills, Management Skills, Cybersecurity, AI Prompting, Data basics
- Goal: Land a stable job and stay relevant in the next 1.5 years
- Available Time: 2 hours per day
- Experience Level: Between beginner and intermediate
- Preferred Learning Style: Videos and hands-on Projects

Include:
- Weekly milestones
- Daily tasks
- Resources
- Projects
- Final outcome

Make it practical and beginner-friendly.
```

*(Thinking.....)*

Claude then asked 3 follow-up questions:

```
Q: What topic do you want to learn?
A: Data Science & ML

Q: What's your current experience level?
A: Some basics

Q: How much time can you dedicate daily?
A: 30–60 min
```

**Total prompts needed: 2**

### Prompt B — With Context

```
Create a 30-day learning roadmap.

Context:
- Current Situation: Student
- Current Skills: Basic Python and Java, Soft Skills, Management Skills, Cybersecurity, AI Prompting, Data basics
- Goal: Land a stable job and stay relevant in the next 1.5 years
- Available Time: 2 hours per day
- Experience Level: Between beginner and intermediate
- Preferred Learning Style: Videos and hands-on Projects

Include:
- Weekly milestones
- Daily tasks
- Resources
- Projects
- Final outcome

Make it practical and beginner-friendly.
```

**Total prompts needed: 1** — Claude built it right away without asking anything.

---

## How Context Changed Everything

| Aspect | Without Context (2 Prompts) | With Context (1 Prompt) |
|--------|----------------------------|------------------------|
| Follow-up Questions | Claude asked 3 questions: topic, experience, daily time | No questions asked — built immediately |
| Roadmap Focus | Broad: Python + AI + Cybersecurity (Career Launchpad) | Focused: Data Science & AI specifically |
| Time Commitment | 2 hrs/day in prompt, but roadmap adjusted to 30-60 min after Q&A | 30-45 min/day — matched from the start |
| Skill Level | Generic beginner, covers everything from scratch | Builds on existing basics, skips what I already know |
| Project Choices | Weather CLI, AI Email Summarizer, Security Analyzer | EDA, Titanic Predictor, Image Classifier, AI App |
| Career Direction | Multiple paths: Jr. Analyst, Security, AI Tools | Targeted: Data Science & AI roles |
| Personalization | A roadmap ABOUT someone like me | A roadmap FOR me |
| Prompts Needed | 2 (baseline + follow-up answers) | 1 (context included upfront) |

The baseline asked questions because it didn't have enough to work with. The prompt with context didn't ask for any question and built it right away because everything Claude needed was already there. That's the difference context makes — it doesn't just improve output, it removes the need for iteration.

---

## The Personalized Roadmap (With Context)

The prompt with context produced a focused **30-Day Data Science & AI Roadmap**:

- **Week 1 (Days 1–7):** Data Foundations & Python Refresh — Numpy, Pandas, data cleaning, visualizations, EDA project
- **Week 2 (Days 8–14):** Machine Learning Essentials — Linear/logistic regression, decision trees, Random Forests, model evaluation, Titanic Survival Predictor
- **Week 3 (Days 15–21):** Deep Learning & Neural Networks — Keras, CNNs, transfer learning, NLP, Hugging Face, Image Classifier
- **Week 4 (Days 22–30):** Real-World AI & Portfolio Project — LLM APIs, prompt engineering, RAG, capstone project, GitHub push, end-to-end AI App

**Final Outcome:** Data fluency (Numpy, Pandas, Matplotlib, Seaborn), ML builder (scikit-learn), deep learning (Keras, transfer learning, Hugging Face), AI APIs (LLMs, RAG), GitHub portfolio (4 projects), clear next steps.

---

## Biggest Insight

On Day 4, chain-of-thought changed the AI's **process** — how it reasons through an answer. On Day 5, context engineering changed the AI's **starting point** — what it knows before it begins.

Same prompt. Same structure. Same request. But with context, Claude didn't need to ask "what topic?" or "how much time?" because the answers were already there. It didn't need to guess my skill level or assume my goals. Every decision in the roadmap — from the focus on Data Science to the 30-45 minute daily commitment to the project choices that build on Python basics — came from context I provided upfront, not from questions asked after.

The baseline asked questions to fill gaps. The prompt with context had no gaps to fill. That's the core difference. Context engineering isn't about writing better prompts — it's about giving the AI enough information that it doesn't need to ask you anything. The prompt was identical both times. The context was the variable.

On Day 2, I saw that structure in the prompt produces structure in the output. On Day 3, I saw that a role changes what the AI considers worth saying. On Day 4, I saw that chain-of-thought makes reasoning traceable. On Day 5, I saw that context eliminates the need for clarification entirely. Four days, four different levers — structure, persona, process, and context. The most powerful one? Context. Because without it, none of the other three matter — the AI is guessing.

---

## Tool of the Day — Sider AI

**What it is:** Sider AI is an AI-powered browser assistant that provides access to multiple AI models, writing assistance, summarization, translation, and productivity features directly inside your browser.

**Setup process:**
1. Open the Sider AI website
2. Install the browser extension
3. Pin the extension to your browser toolbar
4. Open any webpage
5. Use Sider AI to summarize, explain, or interact with content
6. Explore the available AI productivity features

**How it helps with context engineering workflows:** Sider AI sits inside the browser, which means you can pull context from any webpage — documentation, articles, tutorials — and feed it directly into your prompts without switching tabs or copy-pasting across windows. For context engineering specifically, this is useful because the richest context often comes from web sources: current job listings for role-specific context, documentation for technical context, or research papers for domain context. Sider makes it faster to gather and apply that context.

---

## Key Learnings

- **Context engineering is not prompt engineering.** Prompt engineering is about how you phrase the request. Context engineering is about what information you provide before the request. Same prompt with different context = completely different output. The prompt was identical both times — the context was the variable.

- **Context eliminates iteration.** The baseline needed 2 prompts because Claude had to ask follow-up questions. The prompt with context needed 1 because everything was already there. Context doesn't just improve the answer — it removes the need for clarification.

- **The AI asked questions when context was missing.** "What topic do you want to learn?", "What's your experience level?", "How much time can you dedicate?" — these are the gaps. When you fill them upfront, the AI builds immediately. When you don't, the AI asks. Every question Claude asked was information that could have been in the original prompt.

- **Generic context → generic output.** The baseline had general context (student, basic skills, 2 hrs/day) but no specific direction. The result was a broad Career Launchpad covering Python, AI, and Cybersecurity. Specific context → specific output. The prompt with context led to a focused Data Science & AI Roadmap because the goal was clear from the start.

- **Comparing across days:** Day 2 (structured prompt) gave me better output through input structure. Day 3 (role-based prompt) gave me different perspectives through persona. Day 4 (chain-of-thought) gave me verifiable reasoning through process. Day 5 (context engineering) gave me immediate, targeted output through information. Four different levers, four different outcomes — and context is the one that makes the others work better.

- **Context engineering is the foundation of modern AI systems.** RAG (Retrieval-Augmented Generation), AI agents with memory, and multi-turn conversations all rely on context engineering. The better the context, the better the output — regardless of how sophisticated the prompt is.

---

## Screenshots

### Collage Overview

| With Context | Without Context |
|---|---|
| ![Collage - Prompt With Context](Screenshots/With%20Context/CollagePWC.png) | ![Collage - Prompt Without Context](Screenshots/Without%20Context/CollageP-NC.png) |

### Side-by-Side Breakdown

#### 🏷️ Title Slide

|---|---|
| ![Title - Prompt With Context](Screenshots/With%20Context/Title_PWC.png) | ![Title - Prompt Without Context](Screenshots/Without%20Context/Title_P-NC.png) |

#### 📋 The Prompt

| With Context | Without Context |
|---|---|
| ![Prompt with Context](Screenshots/With%20Context/PromptWithContext.png) | ![Prompt without Context](Screenshots/Without%20Context/PromptWithoutContext.png) |

#### 📅 Week 1 — Data Foundations vs Python Power-Up Output

| With Context | Without Context |
|---|---|
| ![Week 1 - Data Foundations and Python Refresh](Screenshots/With%20Context/Week1pwc.png) | ![Week 1 - Python Power-Up](Screenshots/Without%20Context/Week1p-nc.png) |

#### 📅 Week 2 — Machine Learning vs AI and Automation Output

| With Context | Without Context |
|---|---|
| ![Week 2 - Machine Learning Essentials](Screenshots/With%20Context/Week2pwc.png) | ![Week 2 - AI and Automation](Screenshots/Without%20Context/Week2p-nc.png) |

#### 📅 Week 3 — Deep Learning vs Data Skills and Cybersecurity Output

| With Context | Without Context |
|---|---|
| ![Week 3 - Deep Learning and Neural Networks](Screenshots/With%20Context/Week3pwc.png) | ![Week 3 - Data Skills and Cybersecurity](Screenshots/Without%20Context/Week3p-nc.png) |

#### 📅 Week 4 — Real-World AI vs Capstone and Job-Ready Polish Output

| With Context | Without Context |
|---|---|
| ![Week 4 - Real-World AI and Portfolio Project](Screenshots/With%20Context/Week4pwc.png) | ![Week 4 - Capstone and Job-Ready Polish](Screenshots/Without%20Context/Week4p-nc.png) |

#### 🏆 Final Outcome

| With Context | Without Context |
|---|---|
| ![What You will Achieve - With Context](Screenshots/With%20Context/Achievedpwc.png) | ![What You will Achieve - Without Context](Screenshots/Without%20Context/Achievedp-nc.png) |
