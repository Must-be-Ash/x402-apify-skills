# UGC Content Factory — Endpoints & Bodies

x402 paid requests (USDC on Base, `exact`) via any x402 mechanism (private key + client, Coinbase Awal, Agentcash MCP, Sponge MCP…). Apify actorIds use a **tilde**. Run URL pattern: `https://api.apify.com/v2/actors/<actorId>/run-sync-get-dataset-items?maxTotalChargeUsd=0.50` (POST). Apify payTo `0x4aAbE17C239eF71c3A26bA7C2b3e0AeBbfC1DF26`. If a body field is rejected, fetch the actor's live input schema: `GET https://api.apify.com/v2/acts/<actorId>` → `taggedBuilds.latest.buildId` → `GET …/builds/<buildId>` → `.inputSchema`.

> **Apify x402 blockers (observed June 2026):** `run-sync-get-dataset-items` takes 60–90 s; MCP-based paid-fetch tools (Sponge, Agentcash) time out or reject at the facilitator before the actor finishes. The async `/runs` endpoint pays and responds in < 2 s but result retrieval (run status + dataset) requires a regular Apify API key — without one the results are inaccessible. **If Apify is blocked, use the Exa fallback for research (see SKILL.md Step 1 fallback).**

## Research fallback — Exa web search (use when Apify is blocked)

`POST https://stableenrich.dev/api/exa/search` · **$0.01/query** · x402 or MPP · no API key needed.

```json
{
  "query": "TikTok viral AI agent developer content hooks 2026",
  "numResults": 5,
  "contents": { "summary": { "query": "what hooks and formats work for AI developer content on TikTok?" } }
}
```

Run 3–4 queries in parallel: (1) viral hooks/formats for the niche, (2) pain-point messaging for the product, (3) what competitor/adjacent products say that gets engagement. The `contents.summary` field returns an AI-synthesized takeaway per result — extract those instead of raw text to keep context small. Verified working end-to-end June 2026.

## Apify TikTok actors

| Actor | Purpose | Body sketch |
|---|---|---|
| `clockworks~tiktok-trends-scraper` | trending hashtags/sounds/creators/videos | `{ "countryCode": "US", "period": "7", "maxItems": 20 }` |
| `clockworks~tiktok-hashtag-scraper` | videos for niche hashtags | `{ "hashtags": ["skincare"], "resultsPerPage": 20 }` |
| `clockworks~tiktok-scraper` | keyword search (most-liked) | `{ "searchQueries": ["vitamin c serum"], "searchSection": "/video", "sortType": 1, "maxItems": 20 }` |
| `clockworks~free-tiktok-scraper` | engagement ranking + author ctx | `{ "postURLs": ["<videoUrl>…"], "shouldDownloadVideos": false }` |
| `clockworks~tiktok-transcript-extractor` | spoken hooks/scripts | `{ "videos": ["<videoUrl>", "…"] }` |
| `scrape-creators~best-tiktok-transcripts-scraper` | cheaper transcript alt | `{ "videoUrls": ["<videoUrl>"] }` |
| `clockworks~tiktok-video-scraper` | download reference file | `{ "postURLs": ["<bestVideoUrl>"], "shouldDownloadVideos": true }` |

Notes: most clockworks actors accept either `postURLs`/`videos` (explicit URLs) or search params; the **field names vary per actor** — confirm via the live schema if unsure. Engagement metrics (`diggCount`/`shareCount`/`playCount`) live on each item. `clockworks~tiktok-scraper` `sortType:1` ≈ most-liked.

## Video generation — ASK the user which one first (Step 0)

### DEFAULT — Sora 2 (native audio) via BlockRun ✅ verified 2026-06-22
Two-step async. **Net ~$0.84 for 8s** (a $0.84 settles once, despite POST + poll both showing a payment).
1. **POST** `https://blockrun.ai/api/v1/videos/generations` (x402 `exact`, payTo `0xe9030014F5DAe217d0A152f02A043567b16c1aBf`):
   ```json
   { "model": "azure/sora-2", "prompt": "<scene + the spoken line, e.g. ...narrator says: 'CDP wallets just work'. soft ambient music>", "duration_seconds": 8 }
   ```
   → `202 { id, status:"queued", poll_url, ... }`. (Field trap: BlockRun uses `duration_seconds` INT, not `seconds`. A bad body → free 400 *before* any charge.)
2. **Poll** the returned `poll_url` (prepend `https://blockrun.ai`) with **GET, x402, same wallet** every ~30–60s. It 402s until you pay; pay-poll once generation is done (~60–180s) → `200 { status:"completed", data:[{url}] }`. An *unpaid* GET just returns the 402 (no status), so you can't check progress for free — wait ~150s then pay-poll once.
3. **Download `data[0].url` immediately** (MP4 expires). It contains **h264 video + aac audio** (verified) — the spoken line + SFX are baked in. **No TTS / no ffmpeg mux needed.**

### LONGER — Seedance 2 Pro t2v, up to 15s, native audio ✅ verified 2026-06-22
Verified: 15.07s, 9:16, h264+aac, one continuous shot. **~$1.35 at 480p / ~$3.60 at 720p** on Base (resolution drives price). Its result is pulled via **SIWX** (not a paid poll), so your x402 client must support SIWX.
1. **POST** `https://stablestudio.dev/api/generate/seedance/t2v` (x402 `exact`; the body-specific 402 quotes the real price on Base, payTo `0x07F067959297767c887dbfA3C72379c66E82a045`; Solana + Tempo/MPP also offered):
   ```json
   { "prompt": "<scene + spoken line>", "duration": "15", "aspectRatio": "9:16", "outputResolution": "480p" }
   ```
   `duration` STRING `'4'..'15'`; `outputResolution` `480p|720p|1080p`; required: prompt, duration, aspectRatio, outputResolution. → `{ success, jobId, status:"pending", pollUrl }`.
2. **Pull the result via SIWX** (free): GET `pollUrl` (= `https://stablestudio.dev/api/jobs/{jobId}`). The unpaid GET returns a 402 carrying a `sign-in-with-x` challenge (a nonce, ~5-min validity). **Sign the nonce with the same wallet that paid** and resend. Repeat every ~30–60s (~5–6 min; `status` pending→loading(progress%)→`complete`), re-signing a fresh nonce each time.
   - **What is SIWX:** an x402 extension that proves wallet identity by *signature* instead of *payment* (like Sign-In-With-Ethereum) — free. Many x402 v2 clients do it automatically; if yours doesn't, sign the challenge's nonce with the paying wallet and attach it. (This repo's internal tooling: `qa-pay … --method=GET --siwx`.)
3. On `complete`, download `result.videoUrl` (expires ~20min). It already contains audio — no separate voiceover needed.
