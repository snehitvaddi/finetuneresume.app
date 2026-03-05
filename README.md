# Finetune Resume

**AI-powered resume tailoring that actually works.** Paste a job description, get a perfectly tailored resume in seconds — not hours.

[Chrome Web Store](https://chromewebstore.google.com) | [Website](https://www.finetuneresume.app) | [API Docs](docs/api.md)

---

## What is Finetune Resume?

A Chrome extension + API that takes your base resume and a job description, then uses LLMs to intelligently tailor your resume for that specific role. Not just keyword stuffing — real, creative rewriting that makes recruiters think you're purpose-built for their role.

### The Problem

Every job application needs a tailored resume. Doing this manually takes 30-60 minutes per application. Most people either:
- Send the same generic resume everywhere (low response rates)
- Spend hours customizing (burns out fast)
- Use AI tools that just sprinkle keywords in (recruiters notice)

### Our Solution

1. **Install the Chrome extension** — works on LinkedIn, Indeed, Greenhouse, Lever
2. **Browse jobs normally** — extension detects job pages automatically
3. **Click "Tailor Resume"** — paste or highlight the job description
4. **Get a PDF in ~10 seconds** — fully tailored, ATS-optimized, ready to submit

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     CHROME EXTENSION (Manifest V3)              │
│  Detects job pages on LinkedIn/Indeed/Greenhouse/Lever          │
│  User pastes JD → Sends to backend API with auth token         │
└──────────────────────────────┬──────────────────────────────────┘
                               │ POST /api/generate-resume
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                     BACKEND (Next.js 14 on Vercel)              │
│                                                                 │
│  1. Authenticate user (Supabase Auth / Google OAuth)            │
│  2. Fetch user's base resume from DB                            │
│  3. Extract job metadata (company, role, requirements)          │
│  4. Select finetune level (basic / good / super)                │
│  5. LLM tailors resume sections with fallback chain             │
│  6. Teacher-Student Council validates & refines output          │
│  7. Merge tailored sections into base resume JSON               │
│  8. Generate PDF via Reactive Resume                            │
│  9. Save resume + track usage metrics                           │
│                                                                 │
│  Returns: { resumeJSON, pdfUrl, atsScore }                      │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│              REACTIVE RESUME (Self-hosted on Railway)            │
│  Resume JSON → Chromium/Browserless → PDF                       │
│  Supports multiple templates (standard, LaTeX-style, etc.)      │
└─────────────────────────────────────────────────────────────────┘
```

### Tech Stack

| Layer | Technology |
|-------|-----------|
| Extension | Chrome Manifest V3, Vite, React, TypeScript |
| Backend | Next.js 14 (App Router), TypeScript, Vercel Serverless |
| Auth | Supabase Auth (Google OAuth) |
| Database | Supabase PostgreSQL with Row-Level Security |
| LLM | Groq (primary), OpenRouter (fallback), multi-model chain |
| PDF | Reactive Resume (self-hosted fork on Railway) |
| Payments | Stripe / Polar |
| Monorepo | Turborepo + npm workspaces |

---

## Key Design Decisions

These are the engineering choices we made and why. If you're building something similar, this is the interesting stuff.

### 1. Three Finetune Levels

Not every application needs the same depth of tailoring:

| Level | What it does | Use case |
|-------|-------------|----------|
| **Basic** | Keyword optimization only — updates skills, adds 1-2 terms to summary | Quick-apply jobs, already a good fit |
| **Good** | Rewrites summary, tweaks experience bullets with relevant keywords | Most applications |
| **Super** | Full creative rewrite — reframes every experience for the target industry | Dream jobs, career pivots |

**Why?** One-size-fits-all tailoring either under-delivers (just keywords) or over-delivers (full rewrite for a job you're already perfect for). Giving users control over depth means faster results when speed matters and deeper tailoring when it counts.

### 2. LLM Fallback Chain (Not a Single Model)

We don't depend on one LLM provider. The system has a cascading fallback:

```
Groq (GPT OSS 120B, fast inference)
  ↓ fails?
OpenRouter → DeepSeek V3.2 (creative, 30s timeout)
  ↓ fails?
OpenRouter → GPT-5 (last resort, 30s timeout)
```

**Why?** LLM APIs go down. Rate limits hit. A single-provider architecture means your users stare at error screens. Our fallback chain means ~99.9% uptime even when individual providers have issues. Groq is primary because of speed (sub-3s inference for most resumes).

### 3. Bullet Point Engineering

This is where most AI resume tools fail. We enforce strict rules:

- **Exact bullet count preservation** — If your resume has 6 bullets per job, the output has 6. Never reduced, minimum floor of 5.
- **14-18 word requirement per bullet** — Shorter bullets look incomplete, longer ones get cut off in ATS systems.
- **SAR structure for first 1-2 bullets** — Situation → Action → Result storytelling. Remaining bullets focus on technical depth.
- **Varied metrics** — No "improved X by 30%, improved Y by 25%" repetition. We enforce mix of multipliers, absolute numbers, time saved, dollar amounts, and percentages.
- **No AI slop** — Banned words like "spearheaded", "leveraged", "synergized", "orchestrated". Uses natural action verbs instead.

### 4. Teacher-Student Council (Quality Gate)

After the initial LLM generates tailored content, a second pass critiques and refines:

```
Student (Generator) → produces tailored resume
       ↓
Teacher (Critic) → scores each section, flags issues
       ↓
  Score < threshold?
       ↓ yes
Refiner → fixes flagged issues with specific feedback
       ↓
Validator → checks bullet counts, word lengths, completeness
```

**Why?** Single-pass LLM generation has inconsistent quality. The council catches short bullets, missing experiences, weak keyword coverage, and metric repetition before the user ever sees the output.

### 5. Creative Industry Bridging (Super Level)

The hardest part of resume tailoring: career pivoters. Someone moving from fintech to healthtech doesn't just need keywords swapped — they need their experience *reframed*.

Our approach:
- Analyzes target industry terminology from the JD
- Maps candidate's existing work to equivalent problems in the target domain
- Preserves factual anchors (companies, dates, titles) while rewriting narratives
- Creates plausible connections: finance → healthcare finance, retail → patient engagement, logistics → medical supply chain

### 6. Extension Detection, Not Scraping

The Chrome extension doesn't scrape job pages. Users paste or highlight the job description.

**Why?**
- Scraping breaks constantly (LinkedIn changes DOM every few weeks)
- Gets your extension banned from Chrome Web Store
- Triggers bot detection, risks user's LinkedIn account
- Paste/highlight is more reliable and user-controlled

### 7. PDF via Self-Hosted Reactive Resume

We forked [Reactive Resume](https://github.com/AmruthPillai/Reactive-Resume) and self-host it on Railway for PDF generation.

**Why not just generate PDFs directly?**
- Reactive Resume handles complex layouts, multiple templates, and pixel-perfect PDF rendering via Chromium
- Maintaining our own PDF engine for resumes would be a massive effort
- Self-hosting gives us control over uptime and custom templates
- The JSON-in, PDF-out interface is clean and composable

---

## API Documentation

Full API docs: [docs/api.md](docs/api.md)

### Quick Example

```bash
curl -X POST https://www.finetuneresume.app/api/generate-resume \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "jobDescription": "We are looking for a Senior Backend Engineer...",
    "companyName": "Acme Corp",
    "position": "Senior Backend Engineer",
    "finetuneLevel": "super"
  }'
```

Response:
```json
{
  "resume": { "...Reactive Resume JSON schema..." },
  "pdfUrl": "https://storage.example.com/resumes/abc123.pdf",
  "atsScore": 87,
  "usage": {
    "model": "gpt-oss-120b",
    "inputTokens": 4200,
    "outputTokens": 2800,
    "latencyMs": 2340
  }
}
```

---

## Supported Job Sites

| Site | Detection | Status |
|------|-----------|--------|
| LinkedIn | Auto-detect job page URLs | Active |
| Indeed | Auto-detect job page URLs | Active |
| Greenhouse | Auto-detect `boards.greenhouse.io` | Active |
| Lever | Auto-detect `jobs.lever.co` | Active |
| Any site | Manual paste/highlight | Always works |

---

## Cost & Performance

| Metric | Value |
|--------|-------|
| Avg generation time | ~8-12 seconds (Groq primary) |
| Avg tokens per resume | ~4,000 input + ~3,000 output |
| LLM cost per resume | ~$0.01-0.05 |
| Fallback success rate | ~99.9% |
| PDF generation | ~2-4 seconds |

---

## Project Structure

```
finetune-resume/
├── packages/
│   ├── backend/        # Next.js 14 API + Dashboard (Vercel)
│   ├── extension/      # Chrome Extension (Manifest V3, Vite + React)
│   └── shared/         # Shared TypeScript types
├── reactive-resume/    # Forked PDF generator (Railway)
└── supabase/           # Database migrations & functions
```

This is a **monorepo** managed with Turborepo and npm workspaces.

---

## Deployment

| Component | Provider | Why |
|-----------|----------|-----|
| Backend API | Vercel | Zero-config Next.js, serverless scaling |
| Database | Supabase | Postgres + Auth + RLS out of the box |
| PDF Generator | Railway | Docker support, persistent processes |
| Extension | Chrome Web Store | Distribution |
| LLM APIs | Groq + OpenRouter | Speed + redundancy |

---

## FAQ

**Is this open source?**
This is the public documentation and architecture repo. The production source code is in a private repository.

**Can I use the API directly?**
Yes — see [API docs](docs/api.md). You'll need an account and auth token from the Chrome extension or dashboard.

**What LLM models do you use?**
Primary: GPT OSS 120B via Groq. Fallbacks: DeepSeek V3.2 and GPT-5 via OpenRouter. The model is abstracted — users don't need to care which one runs.

**How is this different from ChatGPT + "tailor my resume"?**
- Structured output (not freeform text) that maps to real resume templates
- Bullet count preservation, word length enforcement, metric variety checks
- Teacher-Student quality council catches issues before you see them
- One-click PDF generation, not copy-pasting between tools
- Job site integration — works right where you're browsing

**Do you store my data?**
Resumes are stored in your account (Supabase with Row-Level Security — only you can access your data). We track anonymized usage metrics for cost management. No data is shared with third parties.

---

## License

MIT

---

Built by [Varshith](https://github.com/vsneh) | [finetuneresume.app](https://www.finetuneresume.app)
