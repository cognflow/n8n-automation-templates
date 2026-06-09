# P05 — AI Customer Support Triage

An intelligent customer support triage workflow built with n8n and Claude (Anthropic). Receives inbound support messages via webhook, uses Claude to classify and draft responses as structured JSON, then routes each ticket to the appropriate team destination based on category.
<img width="1245" height="594" alt="Screenshot 2026-06-09 201226" src="https://github.com/user-attachments/assets/7c2c312a-839a-469a-b63f-ec77dc345b39" />

---

## Overview

| Property | Detail |
|----------|--------|
| **Project** | P05 — Cognflow Master Build Roadmap |
| **Type** | AI-Powered Support Automation |
| **Trigger** | Webhook (POST) |
| **AI Model** | `claude-opus-4-5` |
| **Status** | ✅ Complete |

---

## Architecture

```
Webhook (POST /p05-support-triage)
  └── Edit Fields          # Normalize name, email, message
      └── Code Node         # Build Claude API payload with system prompt
          └── HTTP Request  # POST to Anthropic /v1/messages
              └── Code Node # Parse structured JSON from Claude response
                  └── Switch (route on category)
                        ├── billing / refund / cancellation → Gmail (Billing Team)
                        ├── technical                       → Slack (#cognflow-alerts)
                        └── general                         → Gmail (Auto-reply)
                              └── Respond to Webhook
```

---

## Key Pattern — Structured JSON Output from Claude

This project introduces prompting Claude to return a strict JSON object rather than free-form text, enabling deterministic workflow branching on AI-parsed fields.

**System prompt instructs Claude to respond ONLY with:**

```json
{
  "category": "billing | technical | general | refund | cancellation",
  "urgency": "low | medium | high",
  "summary": "one sentence summary of the issue",
  "draft_reply": "a professional, empathetic reply to the customer"
}
```

The parse node strips any markdown code fences Claude may wrap around the JSON before calling `JSON.parse()`.

---

## Routing Logic

| Category | Destination | Purpose |
|----------|-------------|---------|
| `billing` | Gmail | Notify billing team |
| `refund` | Gmail | Notify billing team |
| `cancellation` | Gmail | Notify billing team |
| `technical` | Slack | Alert dev/ops channel |
| `general` | Gmail | Auto-reply to customer |

---

## Nodes

| # | Node | Role |
|---|------|------|
| 1 | Webhook | Receives POST request with name, email, message |
| 2 | Edit Fields | Extracts and normalizes body fields |
| 3 | Code (Build Payload) | Constructs Claude API request with system prompt |
| 4 | HTTP Request | Calls Anthropic `/v1/messages` endpoint |
| 5 | Code (Parse Response) | Strips markdown fences, parses JSON, forwards fields |
| 6 | Switch | Routes on `category` field with 5 named outputs |
| 7 | Gmail (Billing) | Sends email for billing, refund, cancellation tickets |
| 8 | Slack | Posts structured alert for technical issues |
| 9 | Gmail (General) | Sends AI-drafted auto-reply for general enquiries |
| 10 | Respond to Webhook | Returns JSON status confirmation to caller |

---

## Credentials Required

| Credential | Type | Used By |
|------------|------|---------|
| Anthropic API Key | Header Auth (`x-api-key`) | HTTP Request node |
| Gmail OAuth | OAuth2 | Gmail nodes |
| Slack | Slack OAuth | Slack node |

> **Note:** Credentials are stored in n8n and are not embedded in the workflow JSON. Always verify no secrets are present before committing exported workflow files.

---

## Test Payload

```json
{
  "name": "James Okafor",
  "email": "customer@example.com",
  "message": "I was charged twice for my subscription last month and need a refund immediately."
}
```

**Send with curl:**

```bash
curl -X POST "http://localhost:5678/webhook-test/p05-support-triage" \
  -H "Content-Type: application/json" \
  -d '{"name":"James Okafor","email":"customer@example.com","message":"I was charged twice for my subscription last month and need a refund immediately."}'
```

**PowerShell (Windows):**

```powershell
[System.IO.File]::WriteAllText("$env:USERPROFILE\test.json", '{"name":"James Okafor","email":"customer@example.com","message":"I was charged twice for my subscription last month and need a refund immediately."}')

curl.exe -X POST "http://localhost:5678/webhook-test/p05-support-triage" -H "Content-Type: application/json" --data-binary "@$env:USERPROFILE\test.json"
```

---

## Category Test Cases

| Scenario | Expected Route |
|----------|---------------|
| "I was double charged" | `billing` → Gmail |
| "API returning 500 errors since last deploy" | `technical` → Slack |
| "Tell me about enterprise pricing" | `general` → Gmail |
| "I want a full refund" | `refund` → Gmail |
| "I want to cancel my subscription" | `cancellation` → Gmail |

---

## Lessons Learned

- **Structured JSON prompting** — explicit system prompts with schema definitions produce reliable, parseable output from Claude
- **Markdown fence stripping** — Claude may wrap JSON in ` ```json ``` ` blocks; always strip before parsing
- **n8n webhook path whitespace** — a leading space in the webhook path silently breaks registration; verify the exact URL in the node panel
- **Header Auth credential** — Anthropic API key must be configured as a Generic Credential Type → Header Auth with header name `x-api-key`
- **Cross-node data referencing** — use `$('Edit Fields').item.json.fieldname` to access upstream node data after branching

---

## Part of Cognflow Master Build Roadmap

| # | Project | Status |
|---|---------|--------|
| 01 | AI Lead Capture & Response | ✅ Complete |
| 02 | Lead Qualification & Routing | ✅ Complete |
| 03 | AI Client Onboarding Engine | ✅ Complete |
| 04 | Onboarding Email Sequence | ✅ Complete |
| **05** | **AI Customer Support Triage** | **✅ Complete** |
| 06 | Invoice Processing Pipeline | 🔲 Planned |
| 07 | Social Media Content Pipeline | 🔲 Planned |
| 08 | AI Proposal Generator | 🔲 Planned |
| 09 | RAG Business Knowledge Bot | 🔲 Planned |
| 10 | Autonomous AI Sales Agent | 🔲 Planned |

---

**Built by [Cognflow](https://github.com/cognflow) — AI Business Automation**
