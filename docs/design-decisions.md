# Design Decisions & Engineering Log

A deeper dive into the "why" behind Finetune Resume's architecture. Useful if you're building something similar or evaluating our approach.

---

## LLM Prompt Engineering

### Bullet Point Quality Problem

Most AI resume tools produce bullets like:
```
"Spearheaded the development of a cutting-edge microservices platform, enhancing system performance by 40%"
```

This fails for three reasons:
1. "Spearheaded" and "cutting-edge" are AI slop — recruiters recognize it instantly
2. Only percentage metrics — looks robotic when every bullet says "improved X by N%"
3. No story — just action + vague result

### Our Bullet Point Framework

**First 1-2 bullets per job: SAR (Situation-Action-Result) storytelling**
```
GOOD: "Faced with 12,000+ unnecessary field dispatches annually, designed a
       BERT-XGBoost intent classifier (88% F1) that eliminated false dispatches
       and saved $2M in operational costs"

BAD:  "Designed a BERT-XGBoost intent classifier that achieved an 88% F1 score"
```

**Remaining bullets: Technical depth + keyword coverage**
```
GOOD: "Deployed Kubernetes-based auto-scaling with Istio service mesh,
       handling 10M+ daily requests at 99.99% uptime"

BAD:  "Facing scaling issues, deployed Kubernetes..." (unnecessary preamble)
```

### Metric Variety Enforcement

We require at least 3 different metric types per job entry:

1. Multipliers: "3x faster", "10x throughput"
2. Absolute numbers: "Processed 10M+ events daily"
3. Time saved: "Reduced from 6 weeks to 10 days"
4. Dollar amounts: "Cut costs by $200K annually"
5. Error/quality: "From 8% error rate to 0.5%"
6. Percentages: Max 1-2 per job entry (used sparingly)

This makes output look human-written, not AI-generated.

### Banned Words

We explicitly ban common AI resume words:
- spearheaded, enhanced, drove, leveraged, synergized, orchestrated

And enforce natural verbs:
- built, shipped, led, designed, tested, launched, simplified, deployed, scaled, streamlined

---

## LLM Provider Strategy

### Why Not Just Use One Provider?

We ran Groq exclusively for the first few months. Then:
- Groq had a 4-hour outage → 100% of our users couldn't generate resumes
- Rate limits during peak hours (Monday mornings = job application surge)
- Model deprecations with short notice

### Current Fallback Chain

```
Tier: Premium (real job applications)
  1. Groq GPT OSS 120B     — fast (~2s), good quality
  2. OpenRouter DeepSeek V3.2 — creative, 30s timeout
  3. OpenRouter GPT-5       — reliable last resort, 30s timeout

Tier: Standard (sample previews, dashboard demos)
  Same chain, speed > quality
```

The 30-second timeout on OpenRouter prevents cascading delays — if a model is slow, we fail fast and try the next one rather than making the user wait 60+ seconds.

### Why Groq Primary?

Speed. Groq's LPU inference gives us sub-3-second generation for most resumes. Users are browsing job listings — they want results NOW, not in 30 seconds. The speed difference between 3s and 15s is the difference between "this is magic" and "is it broken?"

---

## Teacher-Student Council

### The Problem with Single-Pass Generation

LLM output is inconsistent. Even with detailed prompts, a single pass might:
- Miss 2 out of 5 experience entries
- Generate 3 bullets when the original had 7
- Use only percentage metrics
- Produce bullets with 8 words (too short for ATS)

### Our Solution: Multi-Pass Quality Gate

**Pass 1 — Student (Generator)**
Generates the initial tailored resume. This is fast but imperfect.

**Pass 2 — Teacher (Critic)**
A separate LLM call that scores the output:
- Scores each experience section 1-10
- Checks bullet counts match originals
- Flags metric repetition
- Identifies weak keyword coverage
- Returns structured feedback

**Pass 3 — Refiner (if score < threshold)**
Takes the original output + teacher's feedback and fixes specific issues. This is targeted — it only rewrites flagged sections, not the whole resume.

**Pass 4 — Validator (programmatic)**
Non-LLM validation:
- Bullet count matches expected (from base resume)
- Each bullet is 14-18 words
- All experience IDs are present
- No missing sections

This adds ~3-5 seconds but dramatically improves consistency. The refiner only runs when needed (~30% of generations), so most resumes go through in 2 passes.

---

## Extension Architecture

### Why Paste/Highlight Instead of Scraping?

We initially built DOM scrapers for LinkedIn, Indeed, etc. Problems:
1. LinkedIn changes their class names every 2-3 weeks
2. Indeed uses dynamic rendering that breaks querySelector
3. Greenhouse/Lever have dozens of custom themes
4. Chrome Web Store rejected us for "unauthorized data collection"

Paste/highlight is:
- 100% reliable regardless of site changes
- Acceptable to Chrome Web Store review
- Doesn't risk the user's LinkedIn account
- Works on ANY job site, not just supported ones

### Job Page Detection

The extension still auto-detects when you're on a job page (via URL patterns), but only to show the extension UI — not to scrape content. Detection patterns:

- LinkedIn: `linkedin.com/jobs/view/*`
- Indeed: `indeed.com/viewjob*`
- Greenhouse: `boards.greenhouse.io/*/jobs/*`
- Lever: `jobs.lever.co/*/*`

---

## PDF Generation

### Why Reactive Resume?

Building a PDF renderer for resumes is deceptively hard:
- Multiple column layouts
- Consistent typography across systems
- Proper page breaks (don't split a job entry across pages)
- Print-quality output

Reactive Resume solves all of this with Chromium-based rendering. We self-host a fork on Railway so we control uptime and can add custom templates.

### The JSON Interface

The cleanest part of our architecture: the entire system communicates via Reactive Resume's JSON schema. The LLM outputs JSON, we merge it into the base resume JSON, and Reactive Resume renders it. No HTML/CSS templating on our side.

---

## Database Design Principles

### Row-Level Security Everywhere

Every user-facing table has RLS policies. A user can only read/write their own data. Even if there's a bug in our API, the database itself enforces isolation.

### Usage Tracking Granularity

We track per-request: model used, tokens in/out, latency, finetune level, whether it was a sample preview. This lets us:
- Optimize model selection based on real cost data
- Detect when a provider is degrading (latency trending up)
- Give users accurate usage dashboards
- Separate sample preview costs from real generation costs

---

## Things We Tried and Dropped

**Anthropic Claude as primary LLM** — Great quality but too slow (5-8s) and expensive for our volume. Moved to Groq.

**DOM scraping for job descriptions** — Maintenance nightmare. Switched to paste/highlight.

**Single finetune level** — Users complained "it changed too much" (for easy-apply jobs) and "it didn't change enough" (for dream jobs). Three levels solved this.

**Client-side PDF generation** — Tried jsPDF and html2pdf. Results looked amateur. Server-side Chromium rendering via Reactive Resume is night and day.

**Amending bullet counts freely** — LLMs love to reduce bullets to 3-4. We now enforce exact count matching with a minimum floor of 5.
