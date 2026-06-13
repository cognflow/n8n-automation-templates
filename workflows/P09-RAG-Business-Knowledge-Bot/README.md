# P09 — RAG Business Knowledge Bot

> Part of the [Cognflow n8n Automation Templates](https://github.com/cognflow/n8n-automation-templates) portfolio.

## Overview

A fully in-memory RAG (Retrieval-Augmented Generation) pipeline built in n8n — no vector database required. Fetches Cognflow SOPs from Notion, chunks and embeds the content using Cohere, stores vectors in n8n's static global data, then retrieves the most relevant chunks via cosine similarity to answer business questions using Claude.

Triggered via the n8n Chat UI for a natural conversational experience.

---

## Pipeline Architecture

```
Chat Trigger
→ Fetch Notion SOPs          (HTTP Request — Notion Blocks API)
→ Extract + Chunk Text       (Code — plain text extraction + chunking)
→ Generate Embeddings        (HTTP Request — Cohere embed-english-v3.0, input_type: search_document)
→ Build Vector Store         (Code — store chunks + embeddings in $getWorkflowStaticData('global'))
→ Embed User Question        (HTTP Request — Cohere embed-english-v3.0, input_type: search_query)
→ Similarity Search          (Code — cosine similarity against stored vectors)
→ Ask Claude                 (HTTP Request — Anthropic Claude with retrieved context)
→ Format Response            (Code — clean output for chat display)
```

**Total nodes:** 8

---

## Key Patterns

### Cohere Embeddings via HTTP Request (Free Tier)

Cohere's `embed-english-v3.0` model used for both document and query embeddings. The `input_type` distinction is critical — using the wrong type degrades retrieval quality significantly.

```json
{
  "model": "embed-english-v3.0",
  "texts": ["chunk 1", "chunk 2", "..."],
  "input_type": "search_document"
}
```

For the user question:
```json
{
  "model": "embed-english-v3.0",
  "texts": ["What is the onboarding process?"],
  "input_type": "search_query"
}
```

**Rule:** Documents use `search_document`. Queries use `search_query`. Never mix them.

### Batch Embedding to Avoid Rate Limits

All text chunks are sent in a single Cohere request rather than one request per chunk. This avoids rate limit errors on the free tier.

```javascript
// All chunks in one request — not a loop
const texts = chunks.map(c => c.text);
// Send texts[] to Cohere in one HTTP call
```

### In-Memory Vector Store with Static Data

n8n's `$getWorkflowStaticData('global')` persists data across nodes within a single execution — used as a lightweight vector store without any external DB.

```javascript
const staticData = $getWorkflowStaticData('global');
staticData.vectorStore = chunks.map((chunk, i) => ({
  text: chunk.text,
  embedding: embeddings[i]
}));
```

**Important limitation:** Static data does not persist across separate workflow executions. The RAG pipeline re-fetches and re-embeds on every trigger — acceptable for SOP-sized knowledge bases, not for large corpora.

### Cosine Similarity Search in a Code Node

Similarity search implemented entirely in JavaScript — no external library needed.

```javascript
function cosineSimilarity(a, b) {
  const dot = a.reduce((sum, val, i) => sum + val * b[i], 0);
  const magA = Math.sqrt(a.reduce((sum, val) => sum + val * val, 0));
  const magB = Math.sqrt(b.reduce((sum, val) => sum + val * val, 0));
  return dot / (magA * magB);
}

const scored = vectorStore.map(entry => ({
  text: entry.text,
  score: cosineSimilarity(queryEmbedding, entry.embedding)
}));

const topChunks = scored
  .sort((a, b) => b.score - a.score)
  .slice(0, 3)
  .map(c => c.text)
  .join('\n\n');
```

### Cross-Branch Data Reference

The user's original question is passed forward using `$('NodeName').all()` to reference data from an upstream node across branches.

```javascript
const userQuestion = $('Chat Trigger').all()[0].json.chatInput;
```

---

## Webhook / Trigger

Triggered via the **n8n Chat UI** (Chat Trigger node). No external webhook required.

**Example question:**
```
What is the process for onboarding a new client?
```

---

## Integrations

| Service | Purpose | Auth |
|---------|---------|------|
| Notion API | Fetch SOPs (Blocks API) | Header Auth (Bearer token) |
| Cohere | Text embeddings (free tier) | Header Auth (Bearer token) |
| Claude (Anthropic) | Answer generation with context | Header Auth (x-api-key) |

---

## Credentials Required

- `Notion API` — Bearer token from notion.so/my-integrations
- `Cohere API` — Bearer token from dashboard.cohere.com
- `Anthropic API` — Header Auth, key: `x-api-key`

---

## Notion SOP Page

The pipeline fetches content from the Cognflow SOPs page via the Blocks API:

```
GET https://api.notion.com/v1/blocks/{page_id}/children
Header: Notion-Version: 2022-06-28
```

Plain text is extracted from each block's `rich_text[].plain_text` field.

---

## Design Decisions

**Why no vector DB?**
For a single SOPs page with ~5 sections, an in-memory store is sufficient and keeps the pipeline self-contained within n8n. A vector DB (Pinecone, Qdrant) would be warranted for knowledge bases with hundreds of documents.

**Why Cohere over OpenAI embeddings?**
Cohere's free tier supports `embed-english-v3.0` without a credit card — appropriate for a portfolio build. The model produces high-quality 1024-dimension embeddings comparable to OpenAI's `text-embedding-3-small`.

**Why re-embed on every execution?**
The static data approach re-fetches Notion and re-embeds on each trigger. This ensures the knowledge base is always fresh without needing a cache invalidation strategy. For production, embeddings would be pre-computed and stored externally.

---

## Files

```
P09-RAG-Business-Knowledge-Bot/
├── README.md
└── p09-rag-business-knowledge-bot.json
```

---

## Part of the Cognflow Portfolio

| # | Project |
|---|---------|
| P01 | Lead Capture & CRM Sync |
| P02 | Automated Invoice Generator |
| P03 | AI Client Onboarding Engine |
| P04 | Onboarding Email Sequence |
| P05 | AI Customer Support Triage |
| P06 | Invoice Processing Pipeline |
| P07 | Social Media Content Pipeline |
| P08 | AI Proposal Generator |
| **P09** | **RAG Business Knowledge Bot** ✅ |
| P10 | Autonomous AI Sales Agent |

---

*Built by [Igwe Ogaji Samuel](https://linkedin.com/in/igwe-ogaji) — Cloud & DevOps Engineer | AI Automation Specialist*
*GitHub: [github.com/samklin92](https://github.com/samklin92)*
