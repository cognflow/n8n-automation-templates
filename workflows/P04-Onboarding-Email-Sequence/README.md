\# P04 — Onboarding Email Sequence




\## Overview

A time-based, multi-step email sequence that triggers automatically when a new user signs up via webhook. Each email is AI-generated and personalised by Claude. Introduces the Wait node for time-based logic and engagement-based branching for pro vs free users.



\## Architecture

```

Webhook (signup) → Edit Fields → Code (Email 1) → HTTP Request (Claude) → Gmail (Email 1)

→ Wait 2 Days → Code (Email 2) → HTTP Request (Claude) → Gmail (Email 2)

→ Wait 3 Days → IF (plan\_type = pro?)

&#x20; → True: Code → HTTP Request → Gmail (Email 3a — Power User Tips)

&#x20; → False: Code → HTTP Request → Gmail (Email 3b — Re-engagement)

```



\## Stack

\- \*\*Trigger:\*\* Webhook (POST /onboarding-signup)

\- \*\*AI:\*\* Anthropic Claude (claude-sonnet-4-5)

\- \*\*Delivery:\*\* Gmail

\- \*\*Logic:\*\* Wait node, IF node

\- \*\*Language:\*\* JavaScript (Code nodes)

\- \*\*Platform:\*\* n8n



\## Workflow Nodes

| # | Node | Purpose |

|---|---|---|

| 1 | Webhook | Receives signup payload via POST |

| 2 | Edit Fields | Normalises name, email, plan\_type, company |

| 3 | Code (JS) | Builds Email 1 Claude prompt dynamically |

| 4 | HTTP Request | Calls Anthropic Claude API |

| 5 | Gmail | Sends immediate welcome email |

| 6 | Wait (2 Days) | Pauses workflow execution for 48 hours |

| 7 | Code (JS) | Builds Email 2 Claude prompt |

| 8 | HTTP Request | Calls Anthropic Claude API |

| 9 | Gmail | Sends Getting Started email |

| 10 | Wait (3 Days) | Pauses workflow execution for 72 hours |

| 11 | IF | Routes by plan\_type = pro |

| 12a | Code + HTTP Request + Gmail | Pro users — Power User Tips email |

| 12b | Code + HTTP Request + Gmail | Free users — Re-engagement email |



\## Email Sequence

| Email | Trigger | Audience | Purpose |

|---|---|---|---|

| Email 1 | Immediately on signup | All users | Warm welcome + next step |

| Email 2 | Day 2 | All users | Getting started — one key action |

| Email 3a | Day 5 | Pro users | Power user tips |

| Email 3b | Day 5 | Free users | Re-engagement + upgrade CTA |



\## Webhook Payload

```json

{

&#x20; "name": "John Doe",

&#x20; "email": "john@company.com",

&#x20; "plan\_type": "pro",

&#x20; "company": "Acme Corp"

}

```



\## Setup Instructions

1\. Import the workflow JSON into your n8n instance

2\. Add your credentials:

&#x20;  - Gmail OAuth

&#x20;  - Anthropic API key (in each HTTP Request node header as `x-api-key`)

3\. Set the Webhook path to `onboarding-signup`

4\. For production: change Wait nodes from test values to 2 Days and 3 Days

5\. Activate the workflow

6\. Send POST requests to: `https://your-n8n-domain/webhook/onboarding-signup`



\## Environment Variables

| Variable | Where Used | Description |

|---|---|---|

| `ANTHROPIC\_API\_KEY` | HTTP Request node headers | Anthropic Claude API key |

| Gmail OAuth | Gmail credentials | Google OAuth token |



\## Key Learnings

\- Wait node pauses workflow execution in n8n's database — no memory consumed during wait

\- False branch nodes show "Node was not executed — took a different path" during True branch tests — this is expected behaviour

\- Reference webhook data across long chains using `$('Webhook').item.json.body.fieldname`

\- Test both branches separately by changing `plan\_type` in the payload



\## Author

\*\*Cognflow\*\* — AI Business Automation Agency  

\[cognflow.io](https://cognflow.io) | \[github.com/cognflow](https://github.com/cognflow)

