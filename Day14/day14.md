# Day 14 — AI Job Red Flag Detector 🚩

## Challenge: #60DayClaudeChallenge

Use Claude AI to detect red flags in a REAL job listing that looks "too good to be true" for a fresher/entry-level cybersecurity candidate.

---

## The Prompt I Used

```
You are an AI Red Flag Detector for job seekers.

Analyze the Job Description and Company Information.

Identify:

1. Unrealistic Requirements
- Excessive experience for the role
- Too many skills/responsibilities
- Contradictory expectations

2. Toxic Workplace Signals
- Burnout indicators
- 'Wear many hats'
- 'Fast-paced', 'rockstar', 'hustle culture'
- Poor work-life balance signals

3. Remote Job Authenticity
- Hidden office requirements
- Relocation expectations
- Misleading remote claims

4. Hiring Risks
- Missing salary information
- Vague responsibilities
- Excessive qualifications
- Suspicious hiring practices

5. Company Risks
- Reputation concerns
- Stability concerns
- Growth or layoff indicators

Output:

## Overall Risk Score (0-100)

### Top Red Flags
- List with severity (1-10)

### Positive Signals
- List positives

### Risk Breakdown
| Category | Risk Level |
|-----------|-----------|
| Requirements | |
| Culture | |
| Remote | |
| Hiring | |
| Company | |

### Final Verdict
✅ Apply
⚠️ Apply with Caution
❌ Avoid

### 5 Smart Interview Questions
Generate questions that help validate the identified risks.

---

Job Description:

Junior Cyber Security Analyst — WebKit24
Location: India / Remote (Work From Home)
Type: Full-Time / Permanent
Salary: $10,001 - $50,000/year (per LinkedIn post) / $138,550/year (per Jaabz listing) — CONFLICTING DATA
Experience: 0-2 years (Entry Level / Fresher)
Posted: February 2026

About the Role:
Looking for a motivated and detail-oriented Junior Cyber Security Analyst to join the security team. The candidate will assist in monitoring security systems, identifying vulnerabilities, and responding to potential threats. This role is ideal for someone passionate about cybersecurity and eager to grow in a remote work environment.

Key Responsibilities:
- Monitor and analyze security alerts from various tools like SIEM, IDS/IPS, and firewalls
- Assist in responding to and investigating security incidents
- Support security operations by monitoring systems, identifying threats, and assisting in incident response
- Monitor security events, analyze risks, and respond to incidents to maintain a strong security posture
- Identify vulnerabilities and assist in vulnerability assessments

Required Skills (listed in the job posting):
- SIEM
- QRadar
- GCP (Google Cloud Platform)
- CEH (Certified Ethical Hacker)
- Penetration Testing
- NIST Framework
- Firewalls
- Incident Response
- IDS/IPS
- 0-2 years experience

Benefits:
- Work From Home
- 5-Day Work Week
- Day Shift
- Remote Work

---

Company Information (from the job posting only — YOU research the rest):

WebKit24 — India-based company
- Established in 2014
- Describes itself as a "Digital Marketing Agency" on its own website
- Part of WASEO Consultants
- Core business: SEO, Digital Marketing, App Development, Business Development, Software Development
- NOT a cybersecurity company — their primary business is digital marketing and software development
- Salary data conflicts wildly across different job boards ($10K-$50K on LinkedIn vs $138K on Jaabz)
- Hiring a dedicated Junior Cyber Security Analyst despite being a digital marketing agency
- Has an AmbitionBox page but very limited reviews
- No Glassdoor page specifically for WebKit24

Research WebKit24 yourself — check what kind of company they actually are, their team size, whether they have any cybersecurity clients or operations, their funding, employee reviews, and whether this role makes sense for their business model.

---

My Profile (the candidate evaluating this role):

Name: Devpal Singh Anand
Current Focus: Cybersecurity Analyst and Data Analyst
Location: India
Education: Master of Computer Applications (MCA) at Amity University Online

Experience (1+ year total, all internship/trainee level):
- Data Analytics Intern at AcmeGrade.com (Aug 2023 – Nov 2023)
- Global Service Desk Trainee at NIIT Foundation (Sep 2023 – Nov 2023)
- Cyber Ops at NIIT (2024)

Certifications:
- Cybersecurity Essentials & Networking Fundamentals — Cisco Networking Academy
- Data & AI Fundamentals & Cybersecurity Analyst Path — IBM SkillsBuild
- Data Analytics in Python — AcmeGrade.com
- Global Service Desk 2.0 — NIIT Foundation
- Cyber Ops — NIIT

Skills:
- Python, Scapy, Pandas, NumPy, Matplotlib, OpenCV, PIL, PyPDF2
- Network Packet Analysis, Threat Detection, Incident Response
- Vulnerability Assessment, Security Monitoring
- Data Cleaning & Preprocessing, Predictive Modeling
- ITIL Framework

Gaps:
- No hands-on SIEM experience (Splunk/QRadar/ArcSight)
- No MITRE ATT&CK application experience
- No CompTIA Security+ certification yet
- No malware analysis or forensic investigation experience
- No cloud security experience

Target Salary: ₹4–8 LPA for entry-level cybersecurity roles in India
```

**Source:** This is a **REAL** job listing found on multiple platforms — LinkedIn, Jaabz.com (Job ID 169079), Giglancer.bizoforce.com, Jobbinge.in, and BeBee.com.

---

## The Job I Analyzed

**Junior Cyber Security Analyst — WebKit24**
- 📍 India / Remote (Work From Home)
- 💼 Full-Time / Permanent
- 🎓 0–2 years experience (Entry Level / Fresher)
- 📅 Posted February 2026
- 💰 Salary: CONFLICTING DATA — $10,001–$50,000/yr (LinkedIn) vs $138,550/yr (Jaabz)

### Why This Job Looked "Too Good to Be True"

A fully remote cybersecurity job for freshers with salary claims up to $138K/yr, asking for skills like SIEM, QRadar, GCP, CEH, Pen Testing, and NIST? 🚩🚩🚩

---

## Red Flag Analysis Results

### Overall Risk Score: 78/100 — ❌ AVOID

### 🚩 Top 7 Red Flags

| # | Red Flag | Severity |
|---|----------|----------|
| 1 | **Company is NOT a cybersecurity firm** — WebKit24 is a digital marketing & SEO agency with zero cybersecurity operations | 10/10 |
| 2 | **Wildly conflicting salary data** — 10× discrepancy between job boards for the same role | 9/10 |
| 3 | **Unrealistic skill stack for 0–2 years** — CEH, QRadar, GCP, Pen Testing, NIST = 4–7 year skill set | 9/10 |
| 4 | **Copy-paste job description** — Same responsibility repeated 3 times in different words | 8/10 |
| 5 | **Severely limited employer transparency** — No Glassdoor, minimal AmbitionBox, near-zero review footprint | 8/10 |
| 6 | **Remote claim unverified** — Two Lucknow offices listed, WFH is a "benefit" not confirmed policy | 6/10 |
| 7 | **No security team/infrastructure mentioned** — No SOC, no SIEM vendor, no team size | 7/10 |

### ✅ Positive Signals

- 🏠 Remote-first listed — WFH and 5-day workweek explicitly stated
- ☀️ Day shift only — No night shift or on-call pressure
- 🎯 Entry-level framing — 0–2 years acknowledges freshers
- 📅 Company established 2014 — 10+ years of operation

### 📊 Risk Breakdown

| Category | Risk Level |
|----------|-----------|
| Requirements | 🔴 Very High (90%) |
| Culture | 🟡 Medium (55%) |
| Remote | 🟡 Medium (50%) |
| Hiring | 🔴 Very High (85%) |
| Company | 🔴 High (75%) |

### 👤 Profile Match — Devpal Singh Anand

**Skills Matched (5/10):**
- ✅ Incident Response
- ✅ Security Monitoring
- ✅ Vulnerability Assessment
- ✅ Firewalls (foundational)
- ✅ NIST Framework (awareness)

**Skills Missing:**
- ❌ SIEM/QRadar
- ❌ CEH Certification
- ❌ GCP Cloud Security
- ❌ Penetration Testing
- ~ IDS/IPS (conceptual only)

**Match Rate:** ~40% — But the requirements themselves are unrealistic for 0–2 years

**Key Insight:** Hands-on SIEM (Splunk free trial), CompTIA Security+, and a home lab for network traffic analysis will unlock 3× more relevant openings.

### 📋 What This JD Actually Demands

The title says "Junior" but the skillset screams Senior — they want a 4–7 year professional at fresher pay, working solo at a company that doesn't even do cybersecurity.

### ❓ 5 Smart Interview Questions

1. **On company fit:** WebKit24's website describes you primarily as a digital marketing and SEO agency. Can you walk me through the size of your current security team and the cybersecurity clients this role would support?

2. **On tools & infrastructure:** Which SIEM platform is currently deployed — QRadar, Splunk, or another? And what is the existing alerting infrastructure I'd be monitoring on day one?

3. **On compensation:** Different job boards list this role at very different salary ranges. Can you confirm the actual CTC band and what the structured appraisal cycle looks like?

4. **On remote policy:** The listing says Work From Home — is this fully remote permanently, or is there an expectation to report to the Lucknow office at any frequency? Is this written into the offer letter?

5. **On growth path:** Since CEH and penetration testing are listed as requirements for an entry-level role, does WebKit24 provide study leave, certification reimbursement, or a structured upskilling path?

---

## Screenshots

### Part A — Risk Score & Red Flags
![Risk Score & Red Flags](Screenshots/report-a.png)

### Part B — Positives, Risk Breakdown & Profile Match
![Positives & Risk Breakdown](Screenshots/report-b.png)

### Part C — Interview Questions & Final Verdict
![Interview Questions & Verdict](Screenshots/report-c.png)

---

## Files in This Folder

| File | Description |
|------|-------------|
| `day14.md` | This file — complete write-up with prompt, analysis, and screenshots |
| `day14-red-flag-report.html` | Full dark-themed HTML report (animated) |
| `day14-report-a.html` | Part A: Risk Score + Red Flags |
| `day14-report-b.html` | Part B: Positives + Risk Breakdown + Profile Match |
| `day14-report-c.html` | Part C: Interview Questions + Final Verdict |
| `day14-linkedin-image.html` | LinkedIn overview card (1200×675) |
| `day14-linkedin-post.md` | LinkedIn post text |
| `red-flag-detector.html` | Standalone Red Flag Detector tool (bring your own API key) |
| `Screenshots/` | Split PNG screenshots for LinkedIn/GitHub |

---

## Standalone Tool

The `red-flag-detector.html` is a self-contained tool that lets anyone run the Red Flag Detector with their own API key. Supports:
- **Claude (Anthropic)** — recommended, prompt built & tested with Claude
- OpenRouter (Claude/GPT/Llama)
- OpenAI (GPT-4o)
- Groq (Fast)
- Custom endpoints

Your API key stays in your browser — never sent to any server.

---

## Key Takeaway

> A digital marketing agency hiring a cybersecurity analyst with conflicting salary data, no security infrastructure, and a copy-paste JD is not a genuine opportunity — it's a trap for desperate freshers.

**Look at MSSPs, IT security teams at banks/fintechs, or BPOs with SOC practices instead.**

---

