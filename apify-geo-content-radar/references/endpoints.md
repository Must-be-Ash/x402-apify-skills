# GEO Content Radar — Endpoints & Request Bodies

All calls are **x402 paid requests** (USDC on Base, scheme `exact`) — make them with whatever x402 mechanism the agent has (private key + x402 client, Coinbase Awal, Agentcash MCP, Sponge MCP, etc.). Build the JSON body exactly as shown; only change the values.

**Timeout cause (rail-independent):** the Apify `run-sync-get-dataset-items` endpoint holds the HTTP connection open until the actor finishes; a heavy run (multiple queries × multiple AI engines ≈ 90s+) exceeds the typical client/transport read-timeout (~60–90s) and the request fails — **the Apify actor itself is fine, and this is not specific to any wallet/MCP.** Keep each call small (one AI engine) so it returns in time. (Apify's async run API avoids the long block but generally needs an Apify API key, which breaks the keyless x402 model — so prefer small synchronous calls.)

## Contents
- [Payment facts](#payment-facts)
- [1. Apify Google Search + AI answers](#1-apify-google-search--ai-answers)
- [2. Apify Reddit Scraper Lite](#2-apify-reddit-scraper-lite)
- [3. Apify Tweet Scraper (X) — optional](#3-apify-tweet-scraper-x--optional)
- [4. BlockRun Exa Web Search — optional](#4-blockrun-exa-web-search--optional)
- [Reading Apify responses](#reading-apify-responses)

---

## Payment facts

**Apify (all 3 actors):**
- Method: `POST`. URL pattern: `https://api.apify.com/v2/actors/<actorId>/run-sync-get-dataset-items?maxTotalChargeUsd=0.50`
- x402 accepts: `exact` **and** `upto`, both `eip155:8453` (Base), asset USDC `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`, amount `1000000` (= $1.00), **payTo `0x4aAbE17C239eF71c3A26bA7C2b3e0AeBbfC1DF26`**.
- Use **`exact`**: $1 captured, unused auto-refunded ~1h later. `maxTotalChargeUsd` caps the metered spend (refund ≥ $1 − cap). **Minimum is `0.50`** — lower → HTTP 400 *and still burns the $1 hold*.
- The actor IDs use a **tilde** (`username~name`), not a slash.
- **`run-sync` + `paid_fetch` time out at ~60–90s.** A plain SERP or ONE AI-engine query finishes in time; multi-query × multi-engine batches do NOT (and you still pay the hold). Split the work — one engine per call.

**Exa (first-party — the default):**
- Method: `POST`. URL: `https://api.exa.ai/search`
- x402 accepts: `exact`, Base, USDC, amount `7000` (= $0.007), payTo `0x6d6E695b09861467c7d462f5AAF31cF3540B9192` (Exa's own address).
- Fallback gateways (same shape, $0.01): BlockRun `https://blockrun.ai/api/v1/exa/search` (payTo `0xe9030014…1aBf`), Merit `https://stableenrich.dev/api/exa/search` (payTo `0x325bdF6F…d430`). Use only if first-party Exa fails.

---

## 1. Apify Google Search + AI answers

`actorId = apify~google-search-scraper`
URL: `https://api.apify.com/v2/actors/apify~google-search-scraper/run-sync-get-dataset-items?maxTotalChargeUsd=0.50`

**Required:** `queries` (string; one query per line for multiple).

**Body for the GEO visibility leg — ONE engine per call** (do NOT enable all four at once → timeout). Make four calls, swapping the single engine block each time:
```json
{
  "queries": "best project management software",
  "maxPagesPerQuery": 1,
  "countryCode": "us",
  "languageCode": "en",
  "aiModeSearch": { "enableAiMode": true }
}
```
Then repeat with `"geminiSearch": {"enableGemini": true}`, then `"perplexitySearch": {"enablePerplexity": true, "returnImages": false, "returnRelatedQuestions": false}`, then `"chatGptSearch": {"enableChatGpt": true}`. Read the answer from the matching result field: `aiModeResult.text` + `.sources`, `geminiSearchResult`, `perplexitySearchResult`, `chatGptSearchResult.text` + `.sources`. (`aiOverview` is usually empty — ignore it.)

**Body for the Reddit-discovery leg** (Step 2 — cheaper, no AI add-ons):
```json
{
  "queries": "site:reddit.com best project management software\nsite:reddit.com project management complaints",
  "maxPagesPerQuery": 1,
  "countryCode": "us",
  "languageCode": "en"
}
```

Key add-on fields (all default `false`): `aiModeSearch.enableAiMode`, `geminiSearch.enableGemini`, `perplexitySearch.enablePerplexity`, `chatGptSearch.enableChatGpt`, `copilotSearch.enableCopilot`. Each enabled AI engine adds ~$0.005/query to the metered cost. Other useful fields: `quickDateRange`/`beforeDate`/`afterDate`, `site`, `mobileResults`, `includeUnfilteredResults`. Leave `maximumLeadsEnrichmentRecords` at 0 (lead-enrichment is a separate paid add-on, not needed here).

**What to extract:** per query — organic results, the AI Overviews / AI Mode answer, each enabled engine's AI answer text + **cited source URLs**, and People Also Ask. Match the brand domain + competitor domains against the cited sources.

---

## 2. Apify Reddit Scraper Lite

`actorId = trudax~reddit-scraper-lite`
URL: `https://api.apify.com/v2/actors/trudax~reddit-scraper-lite/run-sync-get-dataset-items?maxTotalChargeUsd=0.50`

**Required:** `proxy` (keep the default Apify residential proxy).

**Body — scrape the threads discovered in Step 2** (preferred; precise + cheap):
```json
{
  "startUrls": [
    { "url": "https://www.reddit.com/r/projectmanagement/comments/XXXX/..." }
  ],
  "skipComments": false,
  "maxItems": 20,
  "maxComments": 30,
  "proxy": { "useApifyProxy": true, "apifyProxyGroups": ["RESIDENTIAL"] }
}
```

**Body — keyword search instead of explicit URLs:**
```json
{
  "searches": ["project management software"],
  "searchPosts": true,
  "searchComments": false,
  "sort": "top",
  "time": "month",
  "maxItems": 20,
  "maxComments": 30,
  "proxy": { "useApifyProxy": true, "apifyProxyGroups": ["RESIDENTIAL"] }
}
```

`sort` enum: `relevance|hot|top|new|rising`. `time` enum: `all|hour|day|week|month|year`. **Prefer `startUrls` from Step 2** — free-text `searches` with a long phrase returns irrelevant junk (verified). Returned fields per item: `dataType, title, body, communityName, url, username, createdAt` — **no `upVotes`/`numberOfComments`** (the *lite* actor omits engagement; rank by Google's Step-2 order or use a non-lite Reddit actor). **What to extract:** titles/bodies + comments → recurring pain points, questions, verbatim phrasing.

---

## 3. Apify Tweet Scraper (X) — optional

`actorId = apidojo~tweet-scraper`
URL: `https://api.apify.com/v2/actors/apidojo~tweet-scraper/run-sync-get-dataset-items?maxTotalChargeUsd=0.50`

**Required:** none, but you MUST supply at least one of `searchTerms` / `startUrls` / `twitterHandles`, and ALWAYS set a small `maxItems` (default is 1000 — too expensive).

```json
{
  "searchTerms": ["project management software", "asana alternative"],
  "sort": "Top",
  "tweetLanguage": "en",
  "maxItems": 50,
  "includeSearchTerms": true
}
```

`sort` enum: `Top|Latest|Latest + Top`. Useful filters: `minimumFavorites`, `minimumRetweets`, `start`/`end` (date range), `onlyVerifiedUsers`. **What to extract:** real-time pain points, hot takes, content angles.

---

## 4. Exa Search (first-party) — optional

URL: `https://api.exa.ai/search` · `POST` · ~$0.007/search · **first-party (Exa's own x402 endpoint — the default; cheaper than the gateways).**

```json
{ "query": "best project management software for agencies", "type": "auto", "numResults": 5 }
```

`type`: `neural|keyword|auto` (default `auto`). Optional: `category` (`company|research paper|news|pdf|github|tweet|personal site|linkedin profile|financial report`), `includeDomains[]`, `excludeDomains[]`, `startPublishedDate`/`endPublishedDate` (ISO-8601). Returns `body.results[]` (title, url, publishedDate, …). Use it to pull top competitor pages for content-gap analysis (read the returned URLs/snippets; fetch full text with a scrape leg only if needed). Fallback gateways if api.exa.ai fails: BlockRun `https://blockrun.ai/api/v1/exa/search`, Merit `https://stableenrich.dev/api/exa/search` (both $0.01, same body).

---

## Reading Apify responses

`run-sync-get-dataset-items` **waits for the run and returns the dataset rows directly as a JSON array** — no polling, no run-id juggling. Each element is one scraped record (a SERP page, a Reddit post, a tweet). If the array is empty or an element carries an `error`/`errorMessage` field, treat the leg as failed and surface it (don't re-pay blindly). The temporary $1 hold reconciles to the real metered cost on the ~1h refund.
