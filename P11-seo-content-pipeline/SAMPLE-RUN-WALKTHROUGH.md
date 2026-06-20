# P11 — Sample Run Walkthrough

> **What this document is:** A representative walkthrough of one execution of the P11 pipeline, showing the data shape at each stage. This is **not a captured live API response** — no Anthropic API key or live WordPress site was available in the build environment for this portfolio project, so this document was constructed by hand to show exactly what each stage produces and consumes, in the same JSON shapes the actual workflow nodes expect and emit.
>
> The one piece of logic that *was* independently tested (not just described) is the markdown-fence-stripping regex used in every Code node after a Claude call — confirmed working against realistic fenced JSON output. That regex pattern is the same one used across the rest of the Cognflow portfolio (P01–P10, M01–M04).
>
> If you're reviewing this as a hiring manager or technical interviewer: ask me to walk through any node's logic live, or to swap the mock competitor-research step for a real SERP API call — the architecture is built so that's a single-node swap, documented in the README.

---

## Stage 1 — Trigger & Keyword Input

**Node:** `Set Target Keyword`

```json
{
  "target_keyword": "best project management software for remote teams",
  "run_started_at": "2026-06-20T09:00:00.000Z"
}
```

---

## Stage 2 — Competitor Research (Simulated)

**Node:** `Simulated Competitor Research [MOCK]`

This is the node flagged as a swap point for a real SERP API integration. Output shape matches what a real scrape would need to produce:

```json
{
  "target_keyword": "best project management software for remote teams",
  "competitor_data": [
    {
      "title": "15 Best Project Management Software for Remote Teams in 2026",
      "url": "https://example-competitor-1.com/pm-software-remote",
      "word_count": 3200,
      "headings": [
        "What to Look for in Remote PM Software",
        "Top Picks Compared",
        "Pricing Breakdown",
        "Integrations to Consider",
        "FAQs"
      ],
      "meta_description": "Compare the 15 best project management tools built for distributed and remote teams, with pricing and feature breakdowns."
    },
    {
      "title": "Best Remote Team PM Tools: Our 2026 Buyer's Guide",
      "url": "https://example-competitor-2.com/buyers-guide",
      "word_count": 2800,
      "headings": [
        "Why Remote Teams Need Different Tools",
        "Our Top 10 Recommendations",
        "Async Collaboration Features",
        "Time Zone Handling",
        "Verdict"
      ],
      "meta_description": "Our 2026 buyer's guide to the best project management software for distributed teams, ranked and compared."
    },
    {
      "title": "10 Project Management Apps That Actually Work for Remote Teams",
      "url": "https://example-competitor-3.com/apps-remote-teams",
      "word_count": 2400,
      "headings": [
        "The Remote Work Problem",
        "App Comparison Table",
        "Best for Small Teams",
        "Best for Enterprise",
        "Final Thoughts"
      ],
      "meta_description": "A practical comparison of 10 project management apps that actually hold up for distributed, remote-first teams."
    }
  ],
  "source": "SIMULATED - replace with live SERP API call"
}
```

---

## Stage 3 — Claude Generates the SEO Brief

**Node:** `Claude: Generate SEO Brief` → `Parse SEO Brief`

The model receives the competitor data and target keyword, and is instructed to return raw JSON only. After fence-stripping and parsing:

```json
{
  "target_keyword": "best project management software for remote teams",
  "recommended_title": "Best Project Management Software for Remote Teams in 2026: A Practical Comparison",
  "recommended_word_count": 2800,
  "required_headings": [
    "What Remote Teams Actually Need from PM Software",
    "Top Tools Compared",
    "Pricing at a Glance",
    "Async and Time Zone Handling",
    "Security and Data Residency for Distributed Teams",
    "FAQs"
  ],
  "related_keywords": [
    "remote team collaboration tools",
    "async project management",
    "distributed team software",
    "time zone scheduling tools"
  ],
  "meta_description": "A practical, up-to-date comparison of the best project management software for remote and distributed teams, covering pricing, async features, and security.",
  "content_gaps": [
    "None of the top competitors address data residency or security compliance for distributed teams operating across multiple jurisdictions — a real concern for teams with EU/US split workforces."
  ]
}
```

This is the most interview-relevant output of the pipeline: the brief doesn't just average the competitors, it explicitly identifies a gap (data residency/compliance) that none of the three competitor articles cover — that's the "Security and Data Residency" heading and the content_gaps field doing real work, not boilerplate.

---

## Stage 4 — Claude Drafts the Article

**Node:** `Claude: Draft Article` → `Parse Article Draft`

Claude receives the full brief and writes the article. Output shape (body abbreviated for this walkthrough — the real output is a full ~2,800-word markdown article):

```json
{
  "article_title": "Best Project Management Software for Remote Teams in 2026: A Practical Comparison",
  "article_body_markdown": "## What Remote Teams Actually Need from PM Software\n\nRemote and distributed teams have different requirements than co-located ones...\n\n[... full ~2,800 word article continues, covering all six required_headings ...]\n\n## Security and Data Residency for Distributed Teams\n\nOne thing most comparison articles skip entirely: if your team spans the EU and US...\n\n[... continues ...]",
  "word_count": 2750,
  "meta_description": "A practical, up-to-date comparison of the best project management software for remote and distributed teams, covering pricing, async features, and security."
}
```

---

## Stage 5 — Self-Review (Agentic Loop)

**Node:** `Claude: Self-Review Draft` → `Parse Self-Review`

This is the agentic step — Claude checks its own draft against the brief's actual requirements rather than the pipeline blindly trusting the first draft.

**Example outcome — first pass, review fails:**

```json
{
  "passes_review": false,
  "missing_headings": [
    "Async and Time Zone Handling"
  ],
  "word_count_ok": true,
  "keywords_ok": true,
  "gap_addressed": true,
  "revision_instructions": "The draft covers five of the six required headings well, but 'Async and Time Zone Handling' was merged into the intro instead of given its own section. Add a dedicated section covering specific async workflows (status updates, handoff documentation) and time zone overlap scheduling, roughly 250-300 words, placed after the pricing section."
}
```

This routes to `Check Revision Count` → `Claude: Revise Draft`, since `passes_review` is `false` and the revision count hasn't hit the cap of 2.

**After one revision — second review:**

```json
{
  "passes_review": true,
  "missing_headings": [],
  "word_count_ok": true,
  "keywords_ok": true,
  "gap_addressed": true,
  "revision_instructions": ""
}
```

`passes_review: true` routes directly to `Prepare WordPress Payload`, skipping further revision.

---

## Stage 6 — WordPress Payload Preparation

**Node:** `Prepare WordPress Payload`

```json
{
  "title": "Best Project Management Software for Remote Teams in 2026: A Practical Comparison",
  "content": "[full revised markdown article]",
  "excerpt": "A practical, up-to-date comparison of the best project management software for remote and distributed teams, covering pricing, async features, and security.",
  "status": "draft",
  "meta": {
    "meta_description": "A practical, up-to-date comparison..."
  },
  "pipeline_status": "published_after_passing_review"
}
```

This is handed to the `WordPress: Create Draft Post [UNVALIDATED]` node — flagged exactly that way in both the workflow JSON and the README, because no live WordPress site existed to confirm this POST actually succeeds against a real installation.

---

## Stage 7 — Run Logging

**Node:** `Prepare Run Log` → `Log Run to Notion`

```json
{
  "run_timestamp": "2026-06-20T09:04:32.000Z",
  "target_keyword": "best project management software for remote teams",
  "article_title": "Best Project Management Software for Remote Teams in 2026: A Practical Comparison",
  "word_count": 2820,
  "pipeline_status": "published_after_passing_review",
  "wordpress_status": "draft (unvalidated - see node notes)"
}
```

This follows the same dual-logging discipline as M03 (Invoice Tracker) — every run leaves an auditable trail regardless of whether the downstream WordPress call succeeded, which matters specifically because the WordPress step is the one unvalidated link in the chain.

---

## What This Walkthrough Demonstrates (and What It Doesn't)

**Demonstrates:**
- Correct JSON contracts between every stage of a 7-stage pipeline
- A genuine agentic loop with a real stop condition (not just a single tool call dressed up as "agentic")
- The same fence-stripping, dual-logging, and JSON-schema-prompting patterns used across the rest of the Cognflow portfolio, applied to a new domain (SEO/content)
- A clearly marked swap point for live scraping and a clearly marked unvalidated point for WordPress

**Does not demonstrate:**
- A live API call to Claude (no key available in the build sandbox)
- A real scrape of live SERP results
- A confirmed-successful POST to a real WordPress installation

If asked in an interview, the honest framing is: *"This is an architecture and logic build, validated by hand-tracing every data contract between stages and unit-testing the parsing logic. The live integrations (SERP API, WordPress) are swap-in points I scoped but didn't have credentials to test end-to-end during this build."*
