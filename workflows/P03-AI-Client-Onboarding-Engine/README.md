\# P03 — AI Client Onboarding Engine



\## Overview

When a new client's status is set to \*\*Active\*\* in the Cognflow Notion CRM, this workflow automatically generates a personalised welcome email via Claude, delivers it via Gmail, creates a client workspace in Notion, and sends an internal alert to Slack — zero manual steps required.



\## Architecture
<img width="1303" height="623" alt="Screenshot 2026-06-07 121210" src="https://github.com/user-attachments/assets/ab6ea25b-8e81-4485-adb0-e4b2bb205dc8" />



\## Stack

\- \*\*Trigger:\*\* Notion database trigger (Client Pipeline)

\- \*\*AI:\*\* Anthropic Claude (claude-sonnet-4-5)

\- \*\*Apps:\*\* Gmail, Notion API, Slack

\- \*\*Platform:\*\* n8n

\- \*\*Language:\*\* JavaScript (Code node)



\## Workflow Nodes

| # | Node | Purpose |

|---|---|---|

| 1 | Notion Trigger | Polls Client Pipeline every minute for updates |

| 2 | IF | Filters for Status = Active only |

| 3 | Code (JavaScript) | Builds dynamic Claude API JSON payload |

| 4 | HTTP Request | Calls Anthropic Claude API |

| 5 | Gmail | Sends personalised welcome email to client |

| 6 | Notion | Creates client workspace page in CRM |

| 7 | Slack | Posts internal alert to #cognflow-alerts |



\## Setup Instructions

1\. Import the workflow JSON into your n8n instance

2\. Add your credentials:

&#x20;  - Notion API (`secret\_...` token with access to Client Pipeline)

&#x20;  - Gmail OAuth

&#x20;  - Anthropic API key (add to HTTP Request header as `x-api-key`)

&#x20;  - Slack Bot Token (`xoxb-...` with `chat:write`, `channels:read` scopes)

3\. Grant the Notion integration access to your Client Pipeline database

4\. Add your Slack bot to the `#cognflow-alerts` channel

5\. Activate the workflow



\## Environment Variables

| Variable | Where Used | Description |

|---|---|---|

| `ANTHROPIC\_API\_KEY` | HTTP Request node header | Anthropic Claude API key |

| Notion Token | Notion credential | Internal integration secret |

| Gmail OAuth | Gmail credential | Google OAuth token |

| Slack Bot Token | Slack credential | xoxb bot token |



\## Key Learnings

\- n8n expressions inside Fixed JSON bodies cause validation errors — use a Code node to build the payload and pass via Expression mode

\- Notion integration requires explicit Content Access granted per database

\- Slack bot must be added to a channel before posting



\## Author

\*\*Cognflow\*\* — AI Business Automation Agency  

\[cognflow.io](https://cognflow.io) | \[github.com/cognflow](https://github.com/cognflow)

