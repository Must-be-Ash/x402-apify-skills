---
name: apify-geo-content-radar
description: Measure a brand's visibility in AI answer engines (Google AI Overviews & AI Mode, ChatGPT, Perplexity, Gemini) and mine Reddit + X for demand-ranked content ideas, producing a one-page GEO (Generative Engine Optimization) + content brief. Use when the user wants to know "do the AI engines recommend / cite us or our competitors?", wants a GEO/AIO visibility audit, or wants data-backed content ideas for a topic, brand, or keyword set. Runs real paid x402 services (Apify actors + optional Exa), paid in USDC on Base via any x402 wallet.
---

# GEO Content Radar

Drop a topic, brand, or keyword set → get back: (1) a **GEO/AIO visibility map** (which sources the AI answer engines cite for the topic, and whether the brand appears), and (2) a **demand-ranked content brief** mined from Reddit + X.

- **Cost:** ~$0.20–0.60 per run (mostly metered Apify usage; see "How this skill executes").
- **Reliability:** Google + Reddit legs are rock-solid; X is an optional booster; Exa is an optional gap-analysis booster.
- **No outward sends** — this skill only reads/researches. Nothing irreversible.

## How this skill executes (read first)

Every leg is a **paid x402 call** (USDC on Base, scheme `exact`). This skill is **payment-method agnostic** — make the call with whatever x402-capable mechanism the running agent has, e.g.:
- a **private key + an x402 client library** (sign the 402 challenge yourself),
- **Coinbase Awal** (the Coinbase agent wallet / Awal CLI),
- **Agentcash MCP**, **Sponge MCP** (`paid_fetch`), or any other x402 wallet/tool.

The skill only specifies the HTTP request (URL, method, JSON body) and the payment facts (x402 `exact`, Base, USDC, the `payTo` in references/endpoints.md). Whatever tool you use, the request/response shape and these findings are identical.

- **Scheme = `exact`.** The Apify gateway advertises both `exact` and `upto`; use `exact`. Each Apify call **captures $1.00 USDC up front, then auto-refunds the unused portion ~1 hour later on-chain** — net cost = the actor's actual metered usage (typically pennies). A temporary $1 hold per Apify call is expected, not a bug.
- **Cost cap:** every Apify run URL carries `?maxTotalChargeUsd=0.50`. **$0.50 is the hard minimum** — a lower value returns HTTP 400 AND still burns the $1 hold (Apify charges before validating). Never set it below 0.50; raise it only for deep runs.
- **⚠️ ONE AI engine (or ~one query) per Apify call.** `run-sync-get-dataset-items` blocks until the run finishes, and most x402 clients time out at ~60–90s. A plain SERP or a single AI-engine query returns in time; **batching multiple queries × multiple AI engines TIMES OUT — and you still pay the $1 hold for nothing.** So: issue a separate paid call per AI engine (AI Mode, then ChatGPT, then Perplexity, then Gemini), each with 1–2 queries. Plain (no-AI) SERP calls can batch several queries safely.
- **A timeout means you probably already paid.** If a call times out, do NOT blindly re-run the same heavy call — reduce scope (fewer engines/queries) first. The $1 hold from the timed-out call refunds with the rest. **There is NO keyless async fallback** — Apify's async `/runs` start is x402-payable but fetching the dataset needs an Apify API key, so async doesn't rescue a keyless wallet.
- **x402-client compatibility:** Apify's challenge is x402 **v2**. A **401 "payment payload invalid / could not be verified by the facilitator"** means your client doesn't interoperate with Apify's v2 facilitator (not a funds problem). Use a client that speaks x402 v2; some clients pay simpler v2 endpoints fine but 401 on Apify — if that happens, switch clients for the Apify legs.
- **Run one leg at a time.** After each paid call, report what came back (row count, key fields, cost) before moving on. If a leg returns an error or an empty/garbage body, STOP and surface it — do not keep paying into a broken leg.
- **Responses can be large** (Reddit/X runs return 80–170KB). Set a small `maxItems` and extract only the fields you need (titles, text, engagement, cited URLs) rather than echoing the whole payload.
- **Pre-flight:** before the first paid call, confirm the agent's wallet has a USDC-on-Base balance. A full run holds up to ~$8 transiently (≈8 Apify calls × $1) before refunds land, even though net settles to well under $1.

Endpoint URLs, exact request bodies, and per-actor input schemas are in **[references/endpoints.md](references/endpoints.md)** — read it before building any request body.

## Step 0 — Collect inputs

Capture from the user (ask only for what's missing):

- **Topic / keyword set** (required) — the subject to audit, e.g. "project management software".
- **Brand** (required for the visibility verdict) — the name + primary domain to look for in AI answers, e.g. "Acme" / `acme.com`.
- **Competitors** (optional) — names/domains to compare against. If omitted, infer them from who the AI engines actually cite.
- **Region / language** (optional) — `countryCode` (e.g. `us`) + `languageCode` (e.g. `en`). Default: US/English.
- **Depth** (optional) — number of core queries (default 3–5) and whether to include the **X** and **Exa** boosters.

Derive 3–5 **core queries** a buyer would actually type (e.g. "best project management software", "notion vs asana", "project management tool for agencies"). These drive every leg.

## Workflow

### Step 1 — AI-answer visibility (Google + AI engines)
Actor `apify~google-search-scraper`. **Make a SEPARATE paid call per AI engine** (one of `aiModeSearch.enableAiMode` / `geminiSearch.enableGemini` / `perplexitySearch.enablePerplexity` / `chatGptSearch.enableChatGpt` at a time — batching them times out; see references/endpoints.md for exact bodies).
Capture, per engine: the AI answer text + the sources it cites, plus the organic SERP and People Also Ask. Record where the **brand and competitors do / do not appear**.
**Audit BOTH framings:** (a) generic/category queries ("best embedded wallet provider") — measures discovery visibility; and (b) branded queries ("is <brand> a good …") — measures branded sentiment. The gap between them is the key GEO finding. (Note: `aiOverview` is often empty in the API; the real AI answers come from the engine add-ons above.)

### Step 2 — Reddit thread discovery (same actor)
In the same step (or a second call), run `site:reddit.com <topic>` queries through `apify~google-search-scraper` to surface the highest-engagement Reddit threads. Collect their URLs for Step 3.

### Step 3 — Reddit pain points
Actor `trudax~reddit-scraper-lite`. **Strongly prefer feeding the thread URLs discovered in Step 2 as `startUrls`** — free-text `searches` with a long phrase returns largely irrelevant posts (verified: a phrase query surfaced r/conspiracy junk). Pull posts + comments → recurring **pain points, questions, and verbatim phrasing**. Caveat: the *lite* actor returns `title/body/url/community` but **no engagement metrics** (`upVotes`/`numberOfComments` are null), so rank by Step-2 discovery order (Google already ranked them) or swap in a non-lite Reddit actor if you need scores.

### Step 4 — X / Twitter listening (optional booster)
Actor `apidojo~tweet-scraper`. Run `searchTerms` on the topic keywords for real-time pain points and content angles. Skip if the user opted out.

### Step 5 — Competitor page gap analysis (optional booster)
Supplement **Exa (first-party, `https://api.exa.ai/search`, ~$0.007/search)** — Exa runs its own x402 endpoint, so prefer it over any gateway. Pull and read the top-ranking competitor pages for content-gap analysis. Skip if the user opted out.

### Step 6 — Synthesize the brief
Produce the one-page brief using **[assets/brief-template.md](assets/brief-template.md)**:
- **GEO/AIO visibility:** per engine, does the brand appear? Which sources/domains does each engine cite and appear to trust? Where is the brand invisible?
- **Demand-ranked pain points & questions** (Reddit + X) with example quotes.
- **Content gaps & ideas:** specific pieces to publish to show up in AI answers, ranked by demand.

## Output

A one-page GEO + content brief (the visibility map, ranked pain points/questions, and a prioritized content-idea list), plus a short research trail: which queries, AI answers, threads, and tweets each conclusion is grounded in, and the total run cost.
