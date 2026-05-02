# Automated CV Screening System

An n8n workflow that automates the first stage of a recruitment pipeline — from CV submission through AI-powered candidate matching — routing qualified candidates to HR and sending automated responses to unmatched applicants.

---

## The Problem

Manually screening CVs is one of the most time-consuming parts of recruitment. For every open role, a recruiter reads dozens of submissions, extracts candidate details, and cross-references them against job requirements before deciding who to advance. This workflow automates that entire process end-to-end.

---

## How It Works

A candidate submits their CV via a web form. The workflow validates the submission, stores the raw CV in cloud storage, parses it using OCR, and passes the structured data to an AI agent. The agent evaluates the candidate against the current open roles database and makes a match decision. Matched candidates are saved to a CRM table and their details are sent to HR. Unmatched candidates receive an automated "no current opening" message.

```
CV Submission Form (webhook trigger)
        ↓
CV Validation
  └─ Rejects malformed or incomplete submissions early
        ↓
Supabase Storage (CV bucket)
  └─ Raw CV file persisted for audit trail
        ↓
OCR / CV Parsing
  └─ Extracts structured text from PDF/image CVs
        ↓
AI Agent (Mistral)
  ├─ Reads parsed CV content
  ├─ Queries Open Roles DB
  └─ Scores candidate against role requirements
        ↓
Candidate matched?
  ├─ Yes → Save to Supabase candidates table
  │         → Send candidate details to HR (email/webhook)
  └─ No  → Send "No current opening" automated response
```

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Workflow orchestration | n8n |
| AI classification | Mistral (via n8n AI Agent node) |
| CV storage | Supabase Storage (CV bucket) |
| Candidate records | Supabase (PostgreSQL table) |
| CV text extraction | OCR node |
| Trigger | n8n Form / Webhook |

---

## Workflow Nodes

| Node | Purpose |
|------|---------|
| CV Submission Form | Receives CV upload and applicant metadata |
| CV Validation | Checks submission completeness before processing |
| Supabase CV Storage Bucket | Persists raw CV file |
| OCR / CV Parsing | Extracts readable text from CV document |
| SO Parser | Structures extracted text into normalized fields |
| AI Agent | Evaluates candidate against open roles |
| Mistral | LLM powering the AI Agent node |
| Open Roles DB | Reference dataset of current vacancies |
| Candidate matched? | Decision branch based on AI Agent output |
| Save matched candidate info | Writes qualified candidate record to Supabase |
| Send details to HR | Notifies HR with candidate summary |
| Send 'No current opening' message | Automated rejection response |

---

## Files

```
├── Automated CV Screening System.json           # n8n workflow (importable)
└── Automated CV Screening System documentation.pdf  # System design and architecture doc
```

---

## Setup

### Prerequisites
- n8n (self-hosted or cloud)
- Supabase project with:
  - A storage bucket for CVs
  - A table for matched candidate records
- Mistral API key
- SMTP or webhook credentials for sending notifications

### Importing the workflow

1. Open your n8n instance
2. Go to **Workflows → Import from file**
3. Select `Automated CV Screening System.json`
4. Update the following in the imported workflow:

**Supabase nodes** — connect your own Supabase credentials under **Settings → Credentials** and update the bucket name and table name to match your project.

**Mistral node** — add your Mistral API key as an n8n credential and relink it to the AI Agent node.

**Open Roles DB node** — replace the reference dataset with your actual open roles. This can be a Supabase table, a Google Sheet, or a static JSON node depending on how you manage vacancies.

**HR notification node** — update the recipient address or webhook URL in the "Send details to HR" node.

**Form trigger URL** — after activating the workflow, n8n will generate a form URL. Share this with applicants or embed it in a careers page.

---

## Design Decisions

**Why Supabase for storage?** Supabase provides both file storage (for raw CVs) and a relational database (for structured candidate records) in a single platform, keeping the workflow's external dependencies minimal.

**Why OCR before the AI agent?** CVs are submitted as PDFs or images. Passing binary files directly to an LLM is unreliable — extracting clean text first via OCR gives the AI agent a consistent, structured input regardless of the CV's original format.

**Why route unmatched candidates immediately?** Holding unmatched applicants in a pending state adds operational overhead. An instant automated response respects the candidate's time and removes the need for manual follow-up on rejections.

---

## License

MIT
