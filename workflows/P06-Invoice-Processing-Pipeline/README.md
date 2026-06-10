# P06 — Invoice Processing Pipeline

## Overview

An automated invoice processing pipeline that accepts PDF invoices via three input methods, extracts structured data using Claude AI, and writes the results to a Google Sheet for tracking.

## Final Architecture (Phase 3)
<img width="1292" height="560" alt="image" src="https://github.com/user-attachments/assets/aeb69fc3-a4e4-4df0-8c3e-811ab195dd3f" />

```
Webhook (POST /invoice-upload)
  → Extract from File (PDF → Text)
  → Code: Build Claude Payload
  → HTTP Request (Claude API)
  → Code: Parse JSON Response
  → Google Sheets (Append or Update Row)
```

## Three Phases Built

| Phase | Trigger | Input Method |
|-------|---------|-------------|
| 1 | Manual Trigger | Read PDF from Docker container disk |
| 2 | Gmail Trigger + Get Message | Email attachment from inbox |
| 3 | Webhook (POST) | Direct PDF file upload via HTTP |

---

## Node Breakdown

### 1. Webhook
- **Method:** POST
- **Path:** `invoice-upload`
- **Option:** Field Name for Binary Data → `data`
- Accepts multipart form-data file upload
- Binary field is named `data0` by n8n internally

### 2. Extract from File
- **Operation:** Extract From PDF
- **Input Binary Field:** `data0`
- Converts binary PDF into raw text string
- Output field: `text`

### 3. Code — Build Claude Payload
Constructs the API request body for Claude:

```javascript
const invoiceText = $input.item.json.text;

const payload = {
  model: "claude-opus-4-5",
  max_tokens: 1000,
  system: `You are an invoice data extraction assistant.
Extract structured data from the invoice text provided and return ONLY a valid JSON object with no markdown, no code fences, no explanation.

Return exactly this schema:
{
  "invoice_number": "",
  "vendor": "",
  "date": "",
  "due_date": "",
  "total_amount": "",
  "currency": "",
  "status": "unpaid"
}`,
  messages: [
    {
      role: "user",
      content: `Extract the invoice data from this text:\n\n${invoiceText}`
    }
  ]
};

return { json: payload };
```

### 4. HTTP Request (Claude API)
- **Method:** POST
- **URL:** `https://api.anthropic.com/v1/messages`
- **Auth:** Generic Credential Type → Header Auth → Anthropic Header Auth (`x-api-key`)
- **Headers:** `anthropic-version: 2023-06-01`
- **Body fields (Expression mode):**
  - `model` → `{{ $json.model }}`
  - `max_tokens` → `{{ $json.max_tokens }}`
  - `system` → `{{ $json.system }}`
  - `messages` → `{{ $json.messages }}`

### 5. Code — Parse JSON Response
Strips markdown fences and parses Claude's response:

```javascript
const rawText = $input.item.json.content[0].text;
const cleaned = rawText.replace(/```json|```/g, '').trim();
const invoice = JSON.parse(cleaned);
return { json: invoice };
```

### 6. Google Sheets — Append or Update Row
- **Document:** Cognflow Invoice Tracker
- **Sheet:** Sheet1
- **Column to match on:** `invoice_number` (prevents duplicates)
- **Mapped columns:**

| Column | Expression |
|--------|-----------|
| invoice_number | `{{ $json.invoice_number }}` |
| vendor | `{{ $json.vendor }}` |
| date | `{{ $json.date }}` |
| due_date | `{{ $json.due_date }}` |
| total_amount | `{{ $json.total_amount }}` |
| currency | `{{ $json.currency }}` |
| status | `{{ $json.status }}` |

---

## Google Sheet Structure

**Sheet name:** Cognflow Invoice Tracker

| invoice_number | vendor | date | due_date | total_amount | currency | status |
|---------------|--------|------|----------|-------------|----------|--------|
| INV-2024-0047 | TechStack Solutions Ltd | 2026-06-05 | 2026-06-19 | 3225 | USD | unpaid |

---

## Key Patterns Learned

### Docker file access
n8n running in Docker cannot access the host filesystem. Copy files into the container:
```powershell
docker exec n8n mkdir -p /home/node/.n8n-files
docker cp "$env:USERPROFILE\Downloads\invoice.pdf" n8n:/home/node/.n8n-files/invoice.pdf
```
n8n restricts file access to `/home/node/.n8n-files`.

### Binary field naming
| Source | Binary Field Name |
|--------|-----------------|
| Read/Write Files from Disk | `data` |
| Gmail Get Message (attachment) | `attachment_0` |
| Webhook file upload | `data0` |

### Gmail attachment extraction
Gmail Trigger alone does not download attachments. Requires a second node:
- **Gmail → Get Message** with **Download Attachments: ON** and **Simplify: OFF**
- Message ID passed as `{{ $json.id }}` from the trigger output

### Google Sheets OAuth setup
Requires Google Cloud Console setup:
1. Enable **Google Sheets API** and **Google Drive API**
2. Create OAuth 2.0 Client ID (Web Application)
3. Add redirect URI: `http://localhost:5678/rest/oauth2-credential/callback`
4. Paste Client ID and Secret into n8n credential

### Duplicate prevention
Set **Column to match on** → `invoice_number` in the Sheets node. n8n checks if the value exists before appending — updates the row if found, appends if not.

### Webhook binary upload test command
```powershell
curl.exe -X POST "http://localhost:5678/webhook-test/invoice-upload" `
  -F "data=@$env:USERPROFILE\Downloads\test-invoice-P06.pdf"
```

---

## Testing

### Phase 1 — Manual
1. Copy PDF into Docker container
2. Click Execute Workflow in n8n

### Phase 2 — Gmail
1. Send email with PDF attachment to `samklinofficial91@gmail.com`
2. Workflow triggers automatically on next poll (every minute)

### Phase 3 — Webhook
```powershell
curl.exe -X POST "http://localhost:5678/webhook/invoice-upload" `
  -F "data=@C:\path\to\invoice.pdf"
```

---

## Cognflow Roadmap Status

| # | Project | Status |
|---|---------|--------|
| 01 | AI Lead Capture & Response | ✅ Complete |
| 02 | Lead Qualification & Routing | ✅ Complete |
| 03 | AI Client Onboarding Engine | ✅ Complete |
| 04 | Onboarding Email Sequence | ✅ Complete |
| 05 | AI Customer Support Triage | ✅ Complete |
| 06 | Invoice Processing Pipeline | ✅ Complete |
| 07 | Social Media Content Pipeline | 🔲 Planned |
| 08 | AI Proposal Generator | 🔲 Planned |
| 09 | RAG Business Knowledge Bot | 🔲 Planned |
| 10 | Autonomous AI Sales Agent | 🔲 Planned |
