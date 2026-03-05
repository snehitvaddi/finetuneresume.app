<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:f97316,50:fb923c,100:fdba74&height=180&section=header&text=Finetune%20Resume&fontSize=42&fontAlignY=32&desc=Stop%20Submitting%20Generic%20Resumes&descAlignY=54&descSize=18&fontColor=ffffff&animation=fadeIn" width="100%" />
</p>

<p align="center">
  <img src="assets/logo.png" width="140" alt="Finetune Resume — AI Otter Mascot" />
</p>

<h3 align="center">AI-tailored resumes for every job application.</h3>
<p align="center"><em>Upload your resume once. We tailor it for every job you apply to, automatically.</em></p>

<br/>

<p align="center">
  <a href="https://www.finetuneresume.app"><img src="https://img.shields.io/badge/Website-finetuneresume.app-f97316?style=for-the-badge&logo=googlechrome&logoColor=white" alt="Website" /></a>&nbsp;
  <a href="https://chromewebstore.google.com/detail/finetune-resume-ai/mpnpoeoabknplmcecjagjahhfgiddgld"><img src="https://img.shields.io/badge/Chrome_Web_Store-Install_Free-4285F4?style=for-the-badge&logo=googlechrome&logoColor=white" alt="Chrome Web Store" /></a>&nbsp;
  <a href="docs/api.md"><img src="https://img.shields.io/badge/API_Docs-Read-1a1a2e?style=for-the-badge&logo=readthedocs&logoColor=white" alt="API Docs" /></a>
</p>

<p align="center">
  <img src="https://img.shields.io/chrome-web-store/rating/mpnpoeoabknplmcecjagjahhfgiddgld?style=flat-square&label=Chrome%20Rating&color=f97316" alt="Chrome Rating" />
  <img src="https://img.shields.io/chrome-web-store/users/mpnpoeoabknplmcecjagjahhfgiddgld?style=flat-square&label=Users&color=f97316" alt="Chrome Users" />
  <img src="https://img.shields.io/badge/cost_per_resume-$0.01-f97316?style=flat-square" alt="Cost" />
  <img src="https://img.shields.io/badge/uptime-99.9%25-brightgreen?style=flat-square" alt="Uptime" />
  <img src="https://img.shields.io/github/license/snehitvaddi/finetuneresume.app?style=flat-square&color=f97316" alt="License" />
</p>

---

## What People Are Saying

> *"This is what me as a job seeker expected... It's like having a personal tailor for your resumes."*
> — **Aditya Sai** &#11088; &#11088; &#11088; &#11088; &#11088;

> *"Used the extension to apply for a job last night... got a call from recruiter to schedule interview!"*
> — **Shreya C.** &#11088; &#11088; &#11088; &#11088; &#11088;

> *"Out of all the tools I tried... provides the most non-AI sounding resume. Magical."*
> — **Nikhil R.** &#11088; &#11088; &#11088; &#11088; &#11088;

> *"SIMPLEST finetuning tool on the market right now. No complicated setup, just works."*
> — **Meera P.** &#11088; &#11088; &#11088; &#11088; &#11088;

<p align="center">
  <strong>5.0</strong> &#11088; on Chrome Web Store
</p>

---

## How It Works

<table>
<tr>
<td align="center" width="25%">
  <h3>1. Upload Resume</h3>
  <p>Upload your PDF or image.<br/>AI extracts everything.</p>
</td>
<td align="center" width="25%">
  <h3>2. Pick a Template</h3>
  <p>Choose from ATS-friendly<br/>professional templates.</p>
</td>
<td align="center" width="25%">
  <h3>3. Browse Jobs</h3>
  <p>Extension auto-detects<br/>job details on LinkedIn.</p>
</td>
<td align="center" width="25%">
  <h3>4. Download PDF</h3>
  <p>AI-tailored resume<br/>ready in one click.</p>
</td>
</tr>
</table>

---

## By The Numbers

<table align="center">
<tr>
<td align="center" width="33%">
  <h1>~10s</h1>
  <p>End-to-end generation</p>
</td>
<td align="center" width="33%">
  <h1>99.9%</h1>
  <p>Uptime via 3-model fallback</p>
</td>
<td align="center" width="33%">
  <h1>$0.01</h1>
  <p>Avg cost per resume</p>
</td>
</tr>
<tr>
<td align="center">
  <h1>14-18</h1>
  <p>Words enforced per bullet</p>
</td>
<td align="center">
  <h1>5+</h1>
  <p>Min bullets per experience</p>
</td>
<td align="center">
  <h1>4</h1>
  <p>Job sites auto-detected</p>
</td>
</tr>
</table>

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                  CHROME EXTENSION  (Manifest V3)                │
│     Detects job pages on LinkedIn / Indeed / Greenhouse / Lever │
│     User pastes JD  -->  Sends to backend API with auth token   │
└──────────────────────────────┬──────────────────────────────────┘
                               │  POST /api/generate-resume
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                  BACKEND  (Next.js 14 on Vercel)                │
│                                                                 │
│   1. Authenticate user  (Supabase Auth / Google OAuth)          │
│   2. Fetch base resume from DB                                  │
│   3. Extract job metadata  (company, role, requirements)        │
│   4. Select finetune level  (basic / good / super)              │
│   5. LLM tailors sections  (3-model fallback chain)             │
│   6. Teacher-Student Council  (critique -> refine -> validate)  │
│   7. Merge tailored JSON into base resume                       │
│   8. Generate PDF via Reactive Resume                           │
│   9. Save + track usage metrics                                 │
│                                                                 │
│   Returns: { resumeJSON, pdfUrl, atsScore }                     │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│             REACTIVE RESUME  (Self-hosted on Railway)           │
│     Resume JSON  -->  Chromium / Browserless  -->  PDF          │
│     Multiple templates  (standard, LaTeX-style, etc.)           │
└─────────────────────────────────────────────────────────────────┘
```

### Tech Stack

<p align="center">
  <img src="https://img.shields.io/badge/Next.js_14-000000?style=for-the-badge&logo=nextdotjs&logoColor=white" />
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white" />
  <img src="https://img.shields.io/badge/React_18-61DAFB?style=for-the-badge&logo=react&logoColor=black" />
  <img src="https://img.shields.io/badge/Supabase-3FCF8E?style=for-the-badge&logo=supabase&logoColor=white" />
  <img src="https://img.shields.io/badge/Vercel-000000?style=for-the-badge&logo=vercel&logoColor=white" />
  <img src="https://img.shields.io/badge/Tailwind_CSS-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white" />
</p>
<p align="center">
  <img src="https://img.shields.io/badge/Chrome_MV3-4285F4?style=for-the-badge&logo=googlechrome&logoColor=white" />
  <img src="https://img.shields.io/badge/Vite-646CFF?style=for-the-badge&logo=vite&logoColor=white" />
  <img src="https://img.shields.io/badge/Turborepo-EF4444?style=for-the-badge&logo=turborepo&logoColor=white" />
  <img src="https://img.shields.io/badge/Railway-0B0D0E?style=for-the-badge&logo=railway&logoColor=white" />
  <img src="https://img.shields.io/badge/Stripe-008CDD?style=for-the-badge&logo=stripe&logoColor=white" />
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" />
</p>

| Layer | Technology |
|:------|:-----------|
| **Extension** | Chrome Manifest V3, Vite 5, React 18, TypeScript |
| **Backend** | Next.js 14 (App Router), Vercel Serverless |
| **Auth** | Supabase Auth (Google OAuth) |
| **Database** | Supabase PostgreSQL + Row-Level Security |
| **LLM** | Groq (primary) -> OpenRouter (DeepSeek V3.2, GPT-5 fallbacks) |
| **PDF** | Reactive Resume (self-hosted fork on Railway) |
| **Payments** | Stripe / Polar |
| **Monorepo** | Turborepo + npm workspaces |

---

## Key Design Decisions

> *The engineering choices that make this different from "just call ChatGPT."*

### 1. Three Finetune Levels

| Level | What It Does | Best For |
|:-----:|:-------------|:---------|
| `basic` | Keyword optimization — skills + summary terms | Quick-apply, already a fit |
| `good` | Summary rewrite + keyword-optimized bullet tweaks | Most applications |
| `super` | Full creative rewrite — reframes every experience for the target industry | Dream jobs, career pivots |

<details>
<summary><strong>Why three levels?</strong></summary>
<br/>
One-size-fits-all tailoring either under-delivers (just keywords) or over-delivers (full rewrite for a job you're already perfect for). Users control the depth — faster results when speed matters, deeper tailoring when it counts.
</details>

---

### 2. LLM Fallback Chain

We never depend on a single provider:

```
  Groq  (GPT OSS 120B  --  fast, sub-3s inference)
    | fails?
    v
  OpenRouter -> DeepSeek V3.2  (creative, 30s timeout)
    | fails?
    v
  OpenRouter -> GPT-5  (last resort, 30s timeout)
```

<details>
<summary><strong>Why not just one model?</strong></summary>
<br/>
LLM APIs go down. Rate limits hit at peak hours (Monday mornings = job application surge). A single-provider architecture means error screens. Our chain means <strong>~99.9% uptime</strong>. The 30-second timeout prevents cascading delays — fail fast, try next.
</details>

---

### 3. Bullet Point Engineering

This is where most AI resume tools fail:

| Rule | What | Why |
|:-----|:-----|:----|
| **Exact count** | Original has 6 bullets -> output has 6. Min floor: 5. | Preserves resume structure |
| **14-18 words** | Every bullet enforced to this range | Shorter = incomplete. Longer = ATS cutoff. |
| **SAR storytelling** | First 1-2 bullets: Situation -> Action -> Result | Recruiters want context, not just claims |
| **Varied metrics** | Mix of multipliers, absolute numbers, $, time saved | All-percentage bullets look AI-generated |
| **No AI slop** | Banned: *spearheaded, leveraged, synergized, orchestrated* | Natural verbs: built, shipped, led, designed |

<details>
<summary><strong>Good vs Bad bullets</strong></summary>
<br/>

**Bad** (AI slop, only percentages):
> "Spearheaded the enhancement of system performance by 40% and drove efficiency improvements by 25%"

**Good** (SAR story, varied metrics):
> "Faced with 12,000+ unnecessary field dispatches annually, designed a BERT-XGBoost classifier (88% F1) that eliminated false dispatches and saved $2M in operational costs"

</details>

---

### 4. Teacher-Student Council

After initial generation, a second LLM pass critiques and refines:

```
  Student (Generator)  -->  produces tailored resume
          |
          v
  Teacher (Critic)     -->  scores sections, flags issues
          |
    Score < threshold?
          | yes
          v
  Refiner              -->  fixes flagged issues with feedback
          |
          v
  Validator            -->  checks bullet counts, word lengths, completeness
```

<details>
<summary><strong>Why multi-pass?</strong></summary>
<br/>
Single-pass LLM generation is inconsistent. The council catches short bullets, missing experiences, weak keyword coverage, and metric repetition <em>before the user ever sees it</em>. The refiner only runs when needed (~30% of generations), so most resumes go through in 2 passes.
</details>

---

### 5. Creative Industry Bridging

For career pivoters — someone moving from fintech to healthtech doesn't just need keywords swapped, they need experience **reframed**:

- Maps existing work to equivalent problems in the target domain
- Preserves factual anchors (companies, dates, titles) while rewriting narratives
- Creates plausible connections: *finance -> healthcare finance, retail -> patient engagement, logistics -> medical supply chain*

---

### 6. Paste, Don't Scrape

The extension **doesn't scrape** job pages. Users paste or highlight the JD.

> Scraping breaks every few weeks (LinkedIn DOM changes), gets extensions banned, and risks user accounts. Paste/highlight is 100% reliable on any job site.

---

### 7. Self-Hosted PDF via Reactive Resume

We forked [Reactive Resume](https://github.com/AmruthPillai/Reactive-Resume) and host on Railway. JSON-in, PDF-out. No HTML/CSS templating on our side.

---

## API Quick Start

Full docs: **[docs/api.md](docs/api.md)**

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

<details>
<summary><strong>Response</strong></summary>

```json
{
  "id": "uuid",
  "resume": { "...Reactive Resume JSON..." },
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

</details>

### Endpoints

| Method | Endpoint | Description |
|:------:|:---------|:------------|
| `POST` | `/api/generate-resume` | Generate tailored resume + PDF |
| `GET` | `/api/resumes` | List all generated resumes |
| `GET` | `/api/resumes/:id` | Get a specific resume |
| `DELETE` | `/api/resumes/:id` | Delete a resume |
| `GET/PUT` | `/api/user/base-resume` | Get or update base resume |
| `POST` | `/api/parse-resume` | Upload PDF -> structured JSON |
| `GET` | `/api/usage` | Usage stats & remaining credits |
| `POST` | `/api/extract-job-metadata` | Parse JD into structured data |
| `GET` | `/api/generate-word/:id` | Export as .docx |

---

## Supported Job Sites

<p align="center">
  <img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" />
  <img src="https://img.shields.io/badge/Indeed-003A9B?style=for-the-badge&logo=indeed&logoColor=white" />
  <img src="https://img.shields.io/badge/Greenhouse-24B47E?style=for-the-badge&logo=greenhouse&logoColor=white" />
  <img src="https://img.shields.io/badge/Lever-1B1D23?style=for-the-badge&logo=lever&logoColor=white" />
  <img src="https://img.shields.io/badge/Any_Site-Paste_&_Highlight-f97316?style=for-the-badge" />
</p>

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

---

## Deployment

| Component | Provider | Why |
|:----------|:---------|:----|
| Backend API | ![Vercel](https://img.shields.io/badge/Vercel-000?style=flat-square&logo=vercel&logoColor=white) | Zero-config Next.js, serverless scaling |
| Database | ![Supabase](https://img.shields.io/badge/Supabase-3FCF8E?style=flat-square&logo=supabase&logoColor=white) | Postgres + Auth + RLS out of the box |
| PDF Generator | ![Railway](https://img.shields.io/badge/Railway-0B0D0E?style=flat-square&logo=railway&logoColor=white) | Docker support, persistent processes |
| Extension | ![Chrome](https://img.shields.io/badge/Chrome_Web_Store-4285F4?style=flat-square&logo=googlechrome&logoColor=white) | Distribution |
| LLM APIs | ![Groq](https://img.shields.io/badge/Groq-F55036?style=flat-square) + ![OpenRouter](https://img.shields.io/badge/OpenRouter-6366F1?style=flat-square) | Speed + redundancy |

---

## FAQ

<details>
<summary><strong>Is this open source?</strong></summary>
<br/>
This is the public architecture and documentation repo. Production source code is in a private repository.
</details>

<details>
<summary><strong>Can I use the API directly?</strong></summary>
<br/>
Yes — see <a href="docs/api.md">API docs</a>. You'll need an account and auth token from the Chrome extension or dashboard.
</details>

<details>
<summary><strong>What LLM models do you use?</strong></summary>
<br/>
Primary: GPT OSS 120B via Groq. Fallbacks: DeepSeek V3.2 and GPT-5 via OpenRouter. The model is abstracted — users don't need to care which one runs.
</details>

<details>
<summary><strong>How is this different from ChatGPT + "tailor my resume"?</strong></summary>
<br/>
<ul>
  <li>Structured output mapping to real resume templates (not freeform text)</li>
  <li>Bullet count preservation, word length enforcement, metric variety checks</li>
  <li>Teacher-Student quality council catches issues before you see them</li>
  <li>One-click PDF generation — no copy-pasting between tools</li>
  <li>Job site integration — works right where you're browsing</li>
</ul>
</details>

<details>
<summary><strong>Do you store my data?</strong></summary>
<br/>
Resumes are stored in your account (Supabase with Row-Level Security — only you can access your data). We track anonymized usage metrics for cost management. No data is shared with third parties.
</details>

---

## Deep Dive

For detailed engineering decisions, prompt strategy, and things we tried & dropped:

**[docs/design-decisions.md](docs/design-decisions.md)** — Full engineering log

---

<p align="center">
  <a href="https://www.finetuneresume.app"><img src="https://img.shields.io/badge/Get_Started_Free-finetuneresume.app-f97316?style=for-the-badge&logo=googlechrome&logoColor=white" /></a>
  &nbsp;&nbsp;
  <a href="https://chromewebstore.google.com/detail/finetune-resume-ai/mpnpoeoabknplmcecjagjahhfgiddgld"><img src="https://img.shields.io/badge/Install_Extension-Chrome_Web_Store-4285F4?style=for-the-badge&logo=googlechrome&logoColor=white" /></a>
</p>

<p align="center">
  Built by <a href="https://www.linkedin.com/in/snehitvaddi/">Snehit Vaddi</a>
</p>

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:f97316,50:fb923c,100:fdba74&height=100&section=footer" width="100%" />
