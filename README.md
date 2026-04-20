# Szponciciel — TikTok AI News Agent

## Description

Szponciciel is a multi-agent content pipeline that autonomously fetches real news, generates TikTok-style video scripts (with a configurable fact/satire ratio), evaluates them in a Writer ↔ Critic loop, and publishes the final videos across a pool of TikTok accounts — each with its own persona, voice, and language.

The project is developed as a study project within the PIAT Jira space.

## Goal

Build a fully automated, self-improving TikTok content factory that:

- Discovers and ranks trending news articles
- Generates persona-specific scripts blending factual and satirical content (`true_fake_ratio`)
- Refines scripts iteratively until a quality threshold is met
- Converts approved scripts into TikTok-ready videos (TTS narration + AI footage + captions)
- Publishes to multiple TikTok accounts via Zernio
- Collects performance metrics and feeds them back into the pipeline to optimize future content

## Agent Workflow

```
┌─────────────────┐
│  News Fetcher   │  Fetches, scrapes, deduplicates, and virality-scores articles
│  (Researcher)   │  Tools: Tavily / Brave Search API + RSS feeds
└────────┬────────┘
         │ ranked article candidates
         ▼
┌─────────────────┐
│     Writer      │  Generates a script per account, parameterized by
│   (Creator)     │  {persona, language, tone, true_fake_ratio, voice}
└────────┬────────┘
         │ draft script
         ▼
┌─────────────────┐
│     Critic      │◄─── rewrites until threshold met or iteration cap hit
│                 │
└────────┬────────┘
         │ approved script
         ▼
┌─────────────────────────────────────────┐
│         Video Content Creator           │
│  TTS (per account voice)                │
│  Video generation (shared per story)    │
│  Caption + hashtag generation           │
└────────────────────┬────────────────────┘
                     │ video package
                     ▼
┌─────────────────────────────────────────┐
│        Output & Distribution            │
│  Upload via Zernio API                  │
│  Attach run metadata {run_id, persona,  │
│  true_fake_ratio, content_tags, ...}    │
└─────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│       Metrics Collection (future)       │
│  TikTok API: views, likes, completion   │
│  Joined on run_id → priors store        │
│  Feeds back into Writer & Critic tuning │
└─────────────────────────────────────────┘
```

Each account runs its own Writer ↔ Critic loop with its own path config. Video footage is generated once per news story and shared; narration and captions are generated per account.

## Results So Far

### Completed Research Spikes

| Spike | Decision |
|---|---|
| **PIAT-17 — Fetching News** | **Tavily** selected as primary browser/search tool. Output schema defined with fields: `title`, `summary`, `url`, `topic`, `source`, `timestamp`, `raw_text`, `language`, `fetch_score`, `paywall`. |
| **PIAT-15 — TikTok Uploading** | **Zernio** selected as the upload platform. Official TikTok Content Posting API does not support fully automated uploads; Zernio bridges this gap. |
| **PIAT-12 — Existing Multi-Agent Solutions** | Surveyed content pipelines, evaluator/critic loops, and automated media production projects for architectural inspiration. |
| **PIAT-11 — Multi-Agent Frameworks** | Evaluated LangGraph, CrewAI, AutoGen, LlamaIndex Workflows. **LangGraph** favored for its native support for cyclic graphs (required by the Creator ↔ Critic loop). |

### In-Progress Research Spikes

| Spike | Status |
|---|---|
| **PIAT-22 — Audio Generation** | Evaluating ElevenLabs vs. OpenAI TTS vs. PlayHT vs. local models (Kokoro, Coqui) for per-persona voices. |
| **PIAT-21 — Caption Generation** | Defining optimal TikTok caption structure (hook + hashtags + CTA) and model selection strategy. |
| **PIAT-20 — Video Generation** | Evaluating Kling AI vs. RunwayML vs. Pika vs. HeyGen for AI footage generation. |

### Open Research Spikes

| Spike | Question |
|---|---|
| **PIAT-26 — Cost Modeling** | End-to-end per-video API cost breakdown across all pipeline stages. |
| **PIAT-25 — Observability Tooling** | Langfuse vs. LangSmith vs. structured logging for LLM tracing and cost tracking. |
| **PIAT-24 — Virality Scoring Model** | Rule-based heuristics vs. LLM-as-judge vs. classifier for ranking article candidates. |
| **PIAT-19 — TikTok Metrics Feedback Loop** | How to correlate post performance back to pipeline parameters via TikTok API. |
| **PIAT-18 — News Deduplication** | Exact URL vs. semantic similarity (embeddings) vs. title hashing strategy. |
| **PIAT-16 — TikTok Account Pool** | Persona design, account rotation strategy, and anti-spam considerations. |

### Implementation Progress

- **PIAT-31** (In Progress) — Zernio platform integration tool
- **PIAT-30** (To Do) — Video Content Creator agent
- **PIAT-29** (To Do) — Writer and Critic agents
- **PIAT-28** (To Do) — Researcher (News Fetcher) agent
- **PIAT-13** (Done) — Agent descriptions written for all workflow nodes
- **PIAT-10** (Done) — Jira scrum board configured
- **PIAT-9** (Done) — Agent workflow graph drawn

### Prototype (this repo)

A working proof-of-concept of the News Fetcher agent is implemented in `src/` as Jupyter notebooks using **LangChain + OpenAI GPT-4o-mini + Tavily**. The second notebook (`test_agent_fixed_schema.ipynb`) adds Pydantic-validated structured output matching the agreed schema.
