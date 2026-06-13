# P07 — Social Media Content Pipeline

## Overview
<img width="1294" height="536" alt="image" src="https://github.com/user-attachments/assets/ba1384e0-724a-4ac9-8584-59c5a835e2bd" />

A fully autonomous content generation and publishing pipeline. Runs on a schedule, reads a content queue from Notion, generates LinkedIn posts via Claude API, publishes directly to LinkedIn, and writes status back to Notion.

This is the first Cognflow workflow to publish autonomously to an external platform.

---

## Architecture

```
Schedule Trigger (Cron)
  → Get Many Database Pages (Notion — Status = Ready)
  → Extract Content Fields (Code)
  → Generate LinkedIn Post (HTTP Request → Claude API)
  → Parse Claude Response (Code)
  → Post to LinkedIn (HTTP Request → LinkedIn UGC API)
  → Update Content Status (Notion — Status = Published)
```

---

## Nodes

| Node | Type | Purpose |
|---|---|---|
| Schedule Trigger | Trigger | Fires daily at 08:00 |
| Get Many Database Pages | Notion | Queries content queue for first Ready row |
| Extract Content Fields | Code | Parses topic, tone, audience, pageId from Notion response |
| Generate LinkedIn Post | HTTP Request | Sends structured prompt to Claude API |
| Parse Claude Response | Code | Extracts and sanitises post text from Claude response |
| Post to LinkedIn | HTTP Request | Posts to LinkedIn via UGC Posts endpoint |
| Update Content Status | Notion | Writes Published status + timestamp + post preview back to Notion |

---

## Notion Content Queue Schema

| Field | Type | Values |
|---|---|---|
| Topic | Title | Short content brief |
| Tone | Select | Professional / Conversational / Technical |
| Target Audience | Text | Description of target reader |
| Status | Select | Draft / Ready / Published / Failed |
| Published At | Date | Written back by n8n on success |
| Post Preview | Text | Claude's generated post stored back |

---

## New Patterns Introduced

- **Scheduled trigger** — time-based execution replacing event-driven flows
- **Notion as a live queue** — read a row, process it, update status back
- **Structured Claude prompting** — topic + tone + audience → long-form LinkedIn post
- **External OAuth publishing** — LinkedIn UGC API (`/v2/ugcPosts`)
- **Conditional write-back** — Notion status updated on success

---

## Setup

### Prerequisites
- n8n running locally (Docker, port 5678)
- Anthropic API key
- LinkedIn Developer App with `w_member_social` scope
- Notion integration token with access to content queue DB

### Credentials Required
| Credential | Type | Used In |
|---|---|---|
| Notion account | Notion API | Get Many Database Pages, Update Content Status |
| Anthropic API key | Header Auth | Generate LinkedIn Post |
| LinkedIn access token | Header Auth (Bearer) | Post to LinkedIn |

### LinkedIn Setup
1. Create a LinkedIn Developer App at [developer.linkedin.com](https://developer.linkedin.com)
2. Add product: **Share on LinkedIn** (`w_member_social`)
3. Add OAuth redirect URL: `http://localhost:5678/rest/oauth2-credential/callback`
4. Generate access token via OAuth flow
5. Replace `PLACEHOLDER_TOKEN` in Post to LinkedIn node Authorization header
6. Replace `YOUR_LINKEDIN_ID` in JSON body with your LinkedIn member URN

### Get Your LinkedIn Member ID
Once you have a valid access token, run:
```bash
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" https://api.linkedin.com/v2/me
```
Use the `id` field as your member URN: `urn:li:person:{id}`

### Notion Content Queue
Database ID: `04da1d07-b783-4d75-b904-68c3c1cbc266`

Add rows with `Status = Ready` to queue posts for publishing.

---

## Key Technical Notes

- Claude model: `claude-sonnet-4-5`
- LinkedIn endpoint: `POST https://api.linkedin.com/v2/ugcPosts`
- Newline characters in Claude response are sanitised before JSON embedding
- Notion filter: `Status equals Ready`, Limit: 1 (processes one post per run)
- Simplify: OFF on Notion nodes to access full property structure

---

## Status
- [x] Phase 1 — Cron + Notion queue read
- [x] Phase 2 — Claude content generation
- [x] Phase 3 — LinkedIn API post (pending live token)
- [x] Phase 4 — Notion status write-back

**Pipeline status: Complete — pending LinkedIn token activation**

---

*Part of the [Cognflow](https://github.com/cognflow) n8n automation portfolio — 10-project roadmap.*
