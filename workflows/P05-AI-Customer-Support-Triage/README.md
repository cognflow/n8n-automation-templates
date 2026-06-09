# P05 — AI Customer Support Triage
Receives a customer support message via webhook, sends it to Claude for structured JSON analysis, then routes it to the correct team based on category.

## Architecture
Webhook → Edit Fields → Code (Build Claude Payload) → HTTP Request (Claude API) → Code (Parse JSON Response) → Switch → billing/refund/cancellation → Gmail | technical → Slack | general → Gmail → Respond to Webhook

## Key Pattern
Claude returns structured JSON with category, urgency, summary, and draft_reply. Workflow branches on category using a Switch node.

## Credentials Required
- Anthropic API (Header Auth)
- Gmail OAuth
- Slack
