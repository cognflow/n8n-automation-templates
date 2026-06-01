# Cognflow — n8n Automation Templates

> Production-ready n8n workflows for AI-powered business automation.
> Built for agencies and businesses that want to automate intelligently.

---

## What is Cognflow?

Cognflow is an AI automation agency that helps businesses eliminate repetitive manual tasks using intelligent, no-code workflows. This repository contains a growing library of production-ready n8n workflow templates — each one built, tested, and ready to deploy.

Whether you are a developer looking to import and extend these workflows or a business owner evaluating automation solutions, this repository gives you a clear picture of what's possible.

---

## Workflow Catalogue

| # | Workflow | Description | Trigger | AI |
|---|----------|-------------|---------|-----|
| 01 | [AI Lead Capture & Response](./AI%20Lead%20Capture%20%26%20Response.json) | Receives a lead via webhook, generates a personalised email using Claude, and sends it via Gmail automatically | Webhook (HTTP POST) | Anthropic Claude |

> More workflows coming soon — customer support automation, invoice processing, lead qualification, and social media content pipelines.

---

## Architecture — AI Lead Capture & Response

```
Web Form / API
     │
     │  HTTPS POST
     ▼
┌─────────────┐     ┌──────────────┐     ┌──────────────────┐     ┌─────────┐     ┌──────────────────┐
│   Webhook   │────▶│  Edit Fields │────▶│  Anthropic Claude│────▶│  Gmail  │────▶│ Respond to       │
│  (Trigger)  │     │  (Map Data)  │     │  (Generate Reply)│     │  (Send) │     │ Webhook (200 OK) │
└─────────────┘     └──────────────┘     └──────────────────┘     └─────────┘     └──────────────────┘
```

**What it does step by step:**

1. **Lead submits a form** — name, email, business name, and message are sent as a JSON payload via HTTPS POST to the n8n webhook endpoint
2. **Edit Fields node** — maps and cleans the incoming data so downstream nodes receive predictable field names
3. **Anthropic Claude API** — receives the lead details and generates a warm, personalised email response in under 2 seconds
4. **Gmail node** — sends the AI-generated email directly to the lead's inbox via Gmail OAuth2
5. **Respond to Webhook** — returns a `200 OK` JSON response to the originating system confirming successful processing

**Data flow:**
```json
Input:
{
  "name": "Ada Okafor",
  "email": "ada@example.com",
  "business": "TechCrush Logistics",
  "message": "I need help automating my customer onboarding"
}

Output (email sent to lead):
  Subject: Re: Your Automation Enquiry
  Body: AI-generated personalised response (< 150 words)

Webhook response:
{
  "status": "success",
  "message": "Thank you, we will be in touch shortly."
}
```

---

## Prerequisites

Before importing any workflow you will need:

- **n8n** — self-hosted via Docker or n8n Cloud
  ```bash
  docker run -it --rm --name n8n -p 5678:5678 n8nio/n8n
  ```
- **Anthropic API key** — get one at [console.anthropic.com](https://console.anthropic.com)
- **Gmail OAuth2 credentials** — set up via Google Cloud Console (see setup guide below)

---

## Setup Guide

### 1. Import the Workflow

1. Open your n8n instance at `http://localhost:5678`
2. Click **New Workflow** → **...** menu → **Import from file**
3. Select the `.json` file from this repository
4. Click **Import**

### 2. Configure Anthropic Credentials

1. Open the **HTTP Request** node in the workflow
2. Under **Send Headers**, find the `x-api-key` header
3. Replace `YOUR_ANTHROPIC_API_KEY_HERE` with your actual Anthropic API key
4. Save

> **Security note:** Never commit your API key to a public repository. Use n8n's built-in credential manager or environment variables in production.

### 3. Configure Gmail OAuth2

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project and enable the **Gmail API**
3. Create **OAuth 2.0 credentials** (Web Application type)
4. Add this redirect URI:
   ```
   http://localhost:5678/rest/oauth2-credential/callback
   ```
5. In n8n, open the **Gmail node** → **Credentials** → **Create New**
6. Paste your **Client ID** and **Client Secret**
7. Click **Sign in with Google** and authorise access

### 4. Test the Workflow

1. Click the **Webhook** node → **Listen for test event**
2. Send a test payload:

**PowerShell:**
```powershell
Invoke-RestMethod -Uri "http://localhost:5678/webhook-test/lead-capture" `
  -Method POST `
  -ContentType "application/json" `
  -Body '{"name":"Ada Okafor","email":"your@email.com","business":"Test Co","message":"I need automation help"}'
```

**curl (Linux/Mac):**
```bash
curl -X POST http://localhost:5678/webhook-test/lead-capture \
  -H "Content-Type: application/json" \
  -d '{"name":"Ada Okafor","email":"your@email.com","business":"Test Co","message":"I need automation help"}'
```

3. Check your inbox — an AI-generated email should arrive within seconds

---

## Environment Variables

For production deployments, store sensitive values as environment variables:

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_API_KEY` | Your Anthropic API key |
| `GMAIL_CLIENT_ID` | Google OAuth2 Client ID |
| `GMAIL_CLIENT_SECRET` | Google OAuth2 Client Secret |

---

## Customisation

**Change the AI model:**
In the HTTP Request node body, update the `model` field:
```json
"model": "claude-opus-4-6"
```

**Change the email tone or length:**
Edit the prompt inside the HTTP Request node body. The current prompt instructs Claude to:
- Address the lead by first name
- Acknowledge their specific problem
- Suggest a discovery call
- Keep the response under 150 words

**Add a CRM integration:**
After the Edit Fields node, add a HubSpot, Airtable, or Notion node to log the lead before Claude generates the response.

---

## Roadmap

Upcoming workflow templates:

- [ ] AI Customer Support — classify and auto-respond to support emails
- [ ] Lead Qualification — score leads by deal value and route to the right team
- [ ] Invoice Processing — extract data from PDF invoices and log to Google Sheets
- [ ] Social Media Content Pipeline — generate and schedule posts from a content brief
- [ ] Onboarding Email Sequence — trigger a multi-step onboarding flow on signup

---

## About Cognflow

Cognflow builds AI-powered automation workflows for businesses that want to work smarter. We design, build, and deploy n8n automations that connect your tools, respond to your leads, and process your data — without manual effort.

**Contact:** [samklinofficial91@gmail.com](mailto:samklinofficial91@gmail.com)
**GitHub:** [github.com/cognflow](https://github.com/cognflow)

---

## License

MIT — feel free to use, modify, and distribute these templates. Attribution appreciated.
