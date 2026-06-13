# Cognflow — n8n Automation Templates

> Production-ready AI-powered workflows built with n8n, Claude, Notion, Gmail, Slack, and more.

**10 workflows. 1 agency. Built from scratch.**

This repository is the complete Cognflow automation portfolio — a library of production-grade n8n workflows covering lead generation, client onboarding, AI content creation, invoice processing, RAG knowledge retrieval, and autonomous sales operations.

Each workflow is fully documented, tested with real data, and ready to import into any n8n instance.

---

## Stack

| Tool | Role |
|------|------|
| [n8n](https://n8n.io) | Workflow engine (self-hosted via Docker) |
| [Anthropic Claude](https://anthropic.com) | AI reasoning, generation, and qualification |
| [Notion API](https://developers.notion.com) | CRM, knowledge base, and logging |
| [Gmail API](https://developers.google.com/gmail) | Automated email sending |
| [Slack API](https://api.slack.com) | Real-time alerts |
| [Cohere](https://cohere.com) | Text embeddings for RAG |
| [Google Docs API](https://developers.google.com/docs) | Document generation |

---

## Workflow Catalogue

### P01 — AI Lead Capture & Response
**Trigger:** Webhook (HTTP POST)

Receives an inbound lead via webhook, passes their details to Claude, generates a personalised response email, and sends it via Gmail — all in under 3 seconds.

**Nodes:** Webhook → Edit Fields → Claude (HTTP Request) → Gmail → Respond to Webhook

**Key pattern:** Claude prompt engineering for personalised email generation. Header Auth for Anthropic API.

---

### P02 — Automated Invoice Generator
**Trigger:** Webhook / Manual

Generates professional invoices automatically based on client and project data. Outputs a structured invoice document and delivers it to the client.

**Key pattern:** Dynamic document generation, structured data mapping.

---

### P03 — AI Client Onboarding Engine
**Trigger:** Notion status change (Active)

Fires automatically when a client is marked Active in the Notion CRM. Sends a multi-step onboarding sequence — welcome email, resource links, and next steps — without any manual effort.

**Nodes:** Webhook → Extract Fields → Claude → Gmail (Welcome Email) → Notion Log

**Key pattern:** `$('Webhook').item.json.body.fieldname` for cross-node data reference. Wait node for stateful sequencing.

---

### P04 — Onboarding Email Sequence
**Trigger:** Webhook

Sends a structured multi-email onboarding sequence triggered on client signup. Each email is timed and personalised using Claude.

**Key pattern:** Wait node for delayed sends. Dynamic Claude API payload construction in a Code node.

---

### P05 — AI Customer Support Triage
**Trigger:** Webhook (inbound support request)

Receives a support message, uses Claude to classify it by category and urgency, generates a draft reply, and routes it via Switch node to the appropriate handler — human escalation or automated response.

**Nodes:** Webhook → Claude Triage → Parse JSON → Switch (category/urgency) → Gmail / Slack

**Key pattern:** Strict JSON schema prompting from Claude (`category`, `urgency`, `summary`, `draft_reply`). Markdown fence stripping before `JSON.parse()`. Switch node routing on extracted fields.

---

### P06 — Invoice Processing Pipeline
**Trigger:** Webhook / Email attachment

Processes incoming invoice data, extracts key fields, logs to Notion, and triggers payment tracking workflows.

**Key pattern:** Structured data extraction, Notion database logging.

---

### P07 — Social Media Content Pipeline
**Trigger:** Schedule / Manual

Generates platform-optimised social media content (LinkedIn, Twitter/X, Instagram) from a brief using Claude, queues posts, and prepares them for publishing.

**Key pattern:** Multi-platform prompt variations from a single brief. Content queue management in Notion.

> ⚠️ LinkedIn publishing pending OAuth token setup.

---

### P08 — AI Proposal Generator
**Trigger:** Webhook (client brief submission)

Takes a client brief and autonomously generates a full 7-section business proposal using Claude. Writes the proposal to a Google Doc, then sends the document link via Gmail to the client.

**Nodes:** Webhook → Extract Brief → Claude (7-section proposal) → Google Docs API (create doc) → Gmail (send link) → Notion (log)

**Key pattern:** Dynamic Google Docs creation via API. Long-form structured generation with Claude. Document URL extraction and embedding in Gmail body.

---

### P09 — RAG Business Knowledge Bot
**Trigger:** n8n Chat UI

A fully in-memory RAG pipeline — no vector database required. Fetches Cognflow SOPs from Notion, chunks and embeds the content via Cohere (`embed-english-v3.0`), stores vectors using `$getWorkflowStaticData('global')`, retrieves the top matching chunks via cosine similarity, and answers business questions using Claude.

**Nodes:** Chat Trigger → Fetch SOPs → Extract + Chunk → Cohere Embed (documents) → Build Vector Store → Cohere Embed (query) → Cosine Similarity Search → Claude → Format Response

**Key patterns:**
- `input_type: search_document` vs `search_query` distinction (Cohere)
- Batch embedding in a single HTTP request to avoid rate limits
- Cosine similarity in a Code node (pure JavaScript, no external library)
- `$getWorkflowStaticData('global')` as in-memory vector store
- `$('NodeName').all()` for cross-branch upstream data reference

---

### P10 — Autonomous AI Sales Agent
**Trigger:** Webhook (lead form submission)

The flagship Cognflow workflow. A fully autonomous sales pipeline — no human input required from lead entry to outreach sent.

Receives a lead, uses Claude to qualify and score them (1–10), routes qualified vs unqualified via Switch, creates a Notion CRM record, fetches internal SOPs as RAG context, generates a personalised outreach email with Claude, sends it via Gmail, updates the CRM with an audit trail, and fires a Slack hot lead alert.

**Nodes:**
```
Webhook
→ Extract Lead Fields
→ Qualify Lead              (Claude — JSON score + intent)
→ Parse Qualification       (Code — fence strip + JSON.parse)
→ Route Lead                (Switch — Qualified ≥ 6 / Unqualified < 6)
   ├── QUALIFIED
   │   → Create Notion Record       (Status: Contacted)
   │   → Fetch SOPs                 (Notion Blocks API)
   │   → Extract SOP Text           (Code — plain text)
   │   → Build Email Prompt         (Code — JSON.stringify body)
   │   → Generate Outreach Email    (Claude — personalised email)
   │   → Parse Email                (Code — subject + body split)
   │   → Send Outreach Email        (Gmail)
   │   → Update Notion Record       (PATCH — audit trail)
   │   → Slack Hot Lead Alert       (#cognflow-alerts)
   └── UNQUALIFIED
       → Create Notion Record - Lost
       → Send Nurture Email         (Gmail)
```

**Key patterns:**
- Agentic lead scoring with structured Claude output
- `JSON.stringify()` in Code node to safely inject dynamic content into HTTP Request bodies
- Notion record creation (POST) + update (PATCH) in same pipeline
- RAG-lite SOP context injection into outreach generation (reuses P09 patterns)
- Full CRM loop: lead in → Notion created → email sent → Notion updated → Slack alerted

---

## Repository Structure

```
cognflow/n8n-automation-templates/
├── README.md
└── workflows/
    ├── P01-AI-Lead-Capture-Response/
    │   ├── README.md
    │   └── p01-ai-lead-capture-response.json
    ├── P03-AI-Client-Onboarding-Engine/
    │   ├── README.md
    │   └── p03-ai-client-onboarding-engine.json
    ├── P04-Onboarding-Email-Sequence/
    │   └── ...
    ├── P05-AI-Customer-Support-Triage/
    │   └── ...
    ├── P06-Invoice-Processing-Pipeline/
    │   └── ...
    ├── P07-Social-Media-Content-Pipeline/
    │   └── ...
    ├── P08-AI-Proposal-Generator/
    │   └── ...
    ├── P09-RAG-Business-Knowledge-Bot/
    │   ├── README.md
    │   └── p09-rag-business-knowledge-bot.json
    └── P10-Autonomous-AI-Sales-Agent/
        ├── README.md
        └── p10-autonomous-ai-sales-agent.json
```

---

## Quick Start

### Prerequisites
- n8n self-hosted via Docker:
```bash
docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n n8nio/n8n
```
- Anthropic API key — [console.anthropic.com](https://console.anthropic.com)
- Notion integration token — [notion.so/my-integrations](https://notion.so/my-integrations)
- Gmail OAuth2 credentials — Google Cloud Console

### Import a Workflow
1. Open n8n at `http://localhost:5678`
2. Click **New Workflow** → **⋮** menu → **Import from file**
3. Select the `.json` file from any workflow folder
4. Configure credentials (API keys, OAuth)
5. Activate and test

### Credentials Setup

**Anthropic (Claude):**
- Authentication: Generic Credential Type → Header Auth
- Header Name: `x-api-key`
- Header Value: your Anthropic API key

**Notion:**
- Authentication: Predefined Credential Type → Notion API
- Token: your Notion integration secret

**Gmail / Slack:**
- Authentication: OAuth2 (follow n8n's built-in OAuth flow)

---

## Test Payload (P10)

```powershell
$body = '{"name":"Adaeze Okonkwo","business":"Okonkwo Logistics Ltd","email":"adaeze@okonkwologistics.com","budget":1500,"pain_point":"We manually send invoices and follow-up emails every week. It takes 2 days of staff time and we keep missing payments. We need this automated.","source":"LinkedIn"}'
[System.IO.File]::WriteAllText("C:\temp\p10_lead.json", $body)
curl.exe -X POST http://localhost:5678/webhook-test/p10-sales-agent -H "Content-Type: application/json" --data-binary "@C:\temp\p10_lead.json"
```

---

## About Cognflow

Cognflow builds AI-powered automation workflows for businesses that want to eliminate manual, repetitive work. We design, build, and deploy n8n automations that connect your tools, respond to your leads, process your documents, and run your operations — without human effort.

**Contact:** samklinofficial91@gmail.com
**LinkedIn:** [linkedin.com/in/igwe-ogaji](https://linkedin.com/in/igwe-ogaji)
**GitHub:** [github.com/samklin92](https://github.com/samklin92)

---

## License

MIT — free to use, modify, and distribute. Attribution appreciated.
