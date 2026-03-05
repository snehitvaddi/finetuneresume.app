# Finetune Resume API Documentation

Base URL: `https://www.finetuneresume.app`

All endpoints require authentication via Bearer token (obtained through the Chrome extension or dashboard login).

---

## Authentication

```
Authorization: Bearer <supabase_access_token>
```

Tokens are obtained via Supabase Auth (Google OAuth). The Chrome extension handles this automatically. For direct API usage, authenticate through the dashboard at `finetuneresume.app` and retrieve your token from the session.

---

## Endpoints

### Generate Tailored Resume

`POST /api/generate-resume`

The core endpoint. Takes a job description and returns a fully tailored resume with PDF.

**Request:**

```json
{
  "jobDescription": "Full job description text...",
  "companyName": "Acme Corp",
  "position": "Senior Software Engineer",
  "finetuneLevel": "super"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `jobDescription` | string | Yes | Full job description text |
| `companyName` | string | Yes | Target company name |
| `position` | string | Yes | Target position/title |
| `finetuneLevel` | `"basic"` \| `"good"` \| `"super"` | No | Tailoring depth (default: `"good"`) |

**Response:**

```json
{
  "id": "uuid",
  "resume": {
    "basics": { "name": "...", "headline": "..." },
    "sections": {
      "summary": { "content": "..." },
      "experience": { "items": [...] },
      "skills": { "items": [...] },
      "projects": { "items": [...] }
    }
  },
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

**Finetune Levels Explained:**

| Level | Description | Avg Time |
|-------|-------------|----------|
| `basic` | Keywords + skills optimization only. Experience bullets untouched. | ~5s |
| `good` | Summary rewrite + keyword-optimized bullet tweaks. Balanced. | ~8s |
| `super` | Full creative rewrite. Every bullet reframed for target role/industry. Teacher-Student quality council runs. | ~12s |

---

### List Resumes

`GET /api/resumes`

Returns all generated resumes for the authenticated user.

**Response:**

```json
[
  {
    "id": "uuid",
    "companyName": "Acme Corp",
    "position": "Senior Software Engineer",
    "finetuneLevel": "super",
    "pdfUrl": "https://...",
    "createdAt": "2026-03-05T10:30:00Z"
  }
]
```

---

### Get Resume

`GET /api/resumes/:id`

Returns a specific resume by ID.

---

### Update Resume

`PATCH /api/resumes/:id`

Update metadata for a resume (e.g., notes, status).

---

### Delete Resume

`DELETE /api/resumes/:id`

Delete a generated resume.

---

### Get/Update Base Resume

`GET /api/user/base-resume` — Retrieve user's base resume JSON

`PUT /api/user/base-resume` — Upload/update base resume

**PUT Request:**

```json
{
  "resume": {
    "basics": { "name": "...", "headline": "..." },
    "sections": { "..." }
  }
}
```

The base resume follows the [Reactive Resume JSON schema](https://docs.rxresu.me). You can export from Reactive Resume and upload directly.

---

### Parse Resume PDF

`POST /api/parse-resume`

Upload a PDF resume and get structured JSON back.

**Request:** `multipart/form-data` with `file` field (PDF)

**Response:**

```json
{
  "resume": {
    "basics": { "..." },
    "sections": { "..." }
  }
}
```

---

### Usage Stats

`GET /api/usage`

Returns usage metrics for the authenticated user.

**Response:**

```json
{
  "remainingCredits": 15,
  "totalGenerated": 42,
  "thisWeek": 3,
  "plan": "free"
}
```

---

### Extract Job Metadata

`POST /api/extract-job-metadata`

Parse a job description into structured metadata.

**Request:**

```json
{
  "jobDescription": "Full job description text..."
}
```

**Response:**

```json
{
  "companyName": "Acme Corp",
  "position": "Senior Software Engineer",
  "location": "San Francisco, CA",
  "salary": "$180k-$220k",
  "requirements": ["5+ years experience", "Python", "AWS"],
  "niceToHave": ["Kubernetes", "ML experience"]
}
```

---

### Generate Word Document

`GET /api/generate-word/:id`

Generate a `.docx` Word document from a saved resume.

**Response:** Binary `.docx` file download.

---

## Error Handling

All errors follow this format:

```json
{
  "error": "Description of what went wrong",
  "code": "ERROR_CODE"
}
```

| Status | Code | Meaning |
|--------|------|---------|
| 401 | `UNAUTHORIZED` | Missing or invalid auth token |
| 403 | `CREDITS_EXHAUSTED` | No remaining credits |
| 404 | `NOT_FOUND` | Resume not found or not owned by user |
| 422 | `VALIDATION_ERROR` | Invalid request body |
| 429 | `RATE_LIMITED` | Too many requests |
| 500 | `LLM_FAILURE` | All LLM providers failed |
| 504 | `TIMEOUT` | Generation took too long |

---

## Rate Limits

- Free tier: 5 resumes/week
- Paid tier: Based on plan

---

## Resume JSON Schema

Resumes follow the [Reactive Resume](https://rxresu.me) JSON format. Key sections:

```
resume
├── basics
│   ├── name
│   ├── headline
│   ├── email, phone, location, url
│   └── picture
├── sections
│   ├── summary        → { content: "HTML string" }
│   ├── experience     → { items: [{ company, position, date, summary }] }
│   ├── education      → { items: [{ institution, degree, date }] }
│   ├── skills         → { items: [{ name, keywords: [] }] }
│   ├── projects       → { items: [{ name, summary, keywords: [] }] }
│   └── ...custom sections
└── metadata
    └── template, theme, layout settings
```

Only `summary`, `experience`, `skills`, and `projects` are modified during tailoring. Everything else (education, contact info, dates, company names, job titles) is preserved exactly as-is.
