# P11 — SEO Content Pipeline (Research → Brief → Agentic Draft → WordPress)

**Status:** Architecture complete and logic-validated. Live scraping and WordPress publishing are unvalidated — see [Honest Status](#honest-status) below.

## What This Is

An n8n workflow that takes a target keyword, researches how competitors currently rank for it, has Claude turn that research into an SEO content brief, writes an article against the brief using an agentic self-review loop, and publishes the result to WordPress as a draft.

Built to close three gaps in the Cognflow automation portfolio that a specific job description called out directly: **SEO workflows**, **web scraping**, and **WordPress posting** — none of which P01–P10 or M01–M04 cover. Rather than building three disconnected demo pieces, this treats them as one pipeline, which is also how they'd realistically work together in production.

## Architecture

```
Trigger (schedule or webhook)
    ↓
Set Target Keyword
    ↓
Competitor Research [MOCK — swap point for live SERP API]
    ↓
Claude: Generate SEO Brief
    ↓
Parse SEO Brief (fence-strip + JSON parse)
    ↓
Claude: Draft Article
    ↓
Parse Article Draft (fence-strip + JSON parse)
    ↓
Claude: Self-Review Draft  ←─────────────┐
    ↓                                    │
Parse Self-Review                        │
    ↓                                    │
Passes Review? ──No──→ Check Revision Count
    │                       ↓
    │                  Max Revisions Hit? ──No──→ Claude: Revise Draft → Parse Revised Draft ──┘
    │                       ↓ Yes
    │                  (flag for human review)
    ↓ Yes
Prepare WordPress Payload
    ↓
WordPress: Create Draft Post [UNVALIDATED]
    ↓
Prepare Run Log
    ↓
Log Run to Notion
```

## Why This Is Agentic, Not Just a Fixed Pipeline

The research → brief → draft sequence is fixed (deliberately — see the Agentic AI Fundamentals notes on when a fixed pipeline beats an agent). The **self-review loop is the agentic part**: Claude evaluates its own draft against the brief's actual requirements, and the pipeline branches based on that evaluation rather than assuming the first draft is good enough. This mirrors the P10 (Autonomous Sales Agent) pattern — a model-driven decision point, not a human-defined branch condition.

It also has the loop-guard the Agentic AI Fundamentals doc flags as a common failure mode: revisions are capped at 2 attempts (`Check Revision Count` node) specifically to prevent thrashing — if the draft still fails review after two revisions, the pipeline routes to a human-review flag instead of looping indefinitely.

## Patterns Reused From the Rest of the Portfolio

- **Markdown fence stripping** before every `JSON.parse()` — same regex used across P01–P10 and M01–M04, independently verified working in this build (see Sample Run Walkthrough)
- **`max_tokens: 4096`** as a floor on the brief and draft generation calls, to avoid silent truncation
- **JSON schema prompting** — every Claude call specifies the exact return shape, not just "return JSON"
- **Header Auth** for the Anthropic API calls, consistent with the rest of the n8n portfolio
- **Dual-logging discipline** from M03 — every run gets logged regardless of whether the WordPress step succeeds, so failures are visible rather than silent

## Honest Status

This is a portfolio architecture build, not a production-validated pipeline, for one specific reason: **no SERP API key and no live WordPress site were available in the build environment.** Rather than fake a successful end-to-end run, two things are explicitly flagged instead of hidden:

1. **`Simulated Competitor Research [MOCK]`** — uses hand-written sample competitor data instead of a live scrape. The node includes inline comments showing exactly what to replace it with (an HTTP Request node against a SERP API) and what output shape the replacement needs to produce. This is a single-node swap, not a redesign.
2. **`WordPress: Create Draft Post [UNVALIDATED]`** — configured against n8n's documented WordPress node schema and the WordPress REST API's expected fields, but never tested against a real WordPress installation. The node's notes field states this explicitly.

What **was** independently validated: the JSON contracts between every stage (hand-traced to confirm each node consumes what the previous node actually produces), and the fence-stripping regex (unit-tested against realistic fenced model output).

See `SAMPLE-RUN-WALKTHROUGH.md` for a full stage-by-stage walkthrough of what one execution produces, with the same honesty framing applied throughout.

## To Make This Production-Ready

1. Get a SERP API key (SerpApi, DataForSEO, or similar) and replace the mock research node with a real HTTP Request call, mapping the API's response into the same `competitor_data` shape the downstream nodes already expect.
2. Set up WordPress Application Password credentials in n8n and run one test post against a real (even staging) WordPress site to confirm the `WordPress: Create Draft Post` node's field mapping is correct.
3. Consider adding a rate-limit guard on the Claude calls if this runs on a schedule against many keywords in sequence, per the AI Automation Fundamentals notes on rate limiting and backoff.
4. Decide on a real "human review" destination for the max-revisions-hit branch (currently just a flag in the payload) — likely a Slack/email alert or a separate Notion view filtered to `pipeline_status: escalated`.

## Files in This Build

- `P11-SEO-Content-Pipeline.json` — the importable n8n workflow
- `SAMPLE-RUN-WALKTHROUGH.md` — stage-by-stage walkthrough of one execution
- `README.md` — this file
