---
name: apify-ugc-content-factory
description: Research what's winning on TikTok in a niche, reverse-engineer a fresh script for a product, then render and voice a brand-new faceless short. Use when the user wants to create faceless short-form video (TikTok/Reels/Shorts) grounded in current trends, asks what is trending on TikTok for a niche, or wants a UGC-style ad/clip generated from proven winners. Runs paid x402 services (Apify TikTok actors plus a video-generation and TTS endpoint), USDC on Base.
---

# UGC Content Factory

Name a product + niche → the skill finds what's surging on TikTok, ranks the winners, pulls their spoken hooks, writes a new script for the product, downloads a reference video, then renders a faceless short with a voiceover.

- **Cost:** ~$1–3 per finished short (Apify research is pennies; the video render is the variable, larger leg).
- **Reliability:** TikTok research + transcripts are rock-solid; the video render is the variable leg.

## How this skill executes (read first)

Every leg is a **paid x402 call** (USDC on Base, scheme `exact`), made with whatever x402 mechanism the agent has — a **private key + x402 client**, **Coinbase Awal** (Awal CLI), **Agentcash MCP**, **Sponge MCP** (`paid_fetch`), etc. The skill gives the request + payment facts; the agent supplies payment. Full URLs/bodies/`payTo`s: **[references/endpoints.md](references/endpoints.md)** — read it first.

**Apify legs:** `POST https://api.apify.com/v2/actors/<actorId>/run-sync-get-dataset-items?maxTotalChargeUsd=0.50` (actorId uses a **tilde**). x402 `exact` = **$1 captured, unused auto-refunded ~1h later** (net = metered usage, usually pennies), payTo `0x4aAbE17C239eF71c3A26bA7C2b3e0AeBbfC1DF26`. `maxTotalChargeUsd` floor is **$0.50**. A single `run-sync` call must finish in ~60-90s or it times out (you still pay the hold) — keep `maxItems` modest. Responses can be large — extract only what you need.

> **⚠️ Apify compatibility warning — read before attempting any Apify leg:**
> Apify uses **x402 version 2**. Two blockers have been observed in practice:
>
> 1. **`run-sync-get-dataset-items` times out with MCP-based x402 tools.** Apify actors take 60–90 s to cold-start and run, but most MCP paid-fetch tools have a shorter internal timeout (no user-configurable option). Even `maxItems: 1–3` does not help because the cold-start dominates. The tool returns "operation timed out" before Apify responds.
>
> 2. **The async `/runs` endpoint starts the job quickly (< 2 s response) and works as a payment**, but Apify locks all result retrieval (run status, dataset items) behind a regular API key. Without an API key you cannot poll status or fetch results after paying. This makes the async approach a dead end unless you have an Apify token.
>
> **If your x402 client returns a 401 "payment payload invalid or could not be verified by the facilitator" error on Apify** — this means your client does not interoperate with Apify's specific x402 v2 facilitator. Other x402 v2 endpoints (Xona, Deepgram, stableenrich, etc.) may still work fine with the same client; the incompatibility is Apify-facilitator-specific, not a global x402 v2 problem.
>
> **Fallback for the research legs:** Replace all Apify TikTok scraper calls with **Exa web search via stableenrich.dev** (`POST https://stableenrich.dev/api/exa/search`, $0.01/query). Run 3–4 targeted searches on TikTok viral content patterns for the niche + product messaging angles. This gives enough signal to write a quality script and is compatible with standard x402 clients. See Step 1 (fallback) below.

**Video (the costly leg — ASK the user which option before rendering).** No lip-sync/talking-avatar endpoint exists, so the output is a faceless clip whose **audio the video model generates itself from your prompt** — put the spoken line + sound/music in the prompt; there is **no separate voiceover step**. Single-call coherent AI video caps ~15s (that's the ceiling for one clip).
- **DEFAULT — Sora 2, ~8–12s, native audio, ~$0.84.** Simplest: POST pays, then poll the result with a normal x402 payment. Cheapest good option.
- **LONGER — Seedance 2 Pro, up to 15s, native audio, 720p, ~$3.60.** Use only when you need 13–15s (for ≤12s, Sora 2 is cheaper and simpler). To cut cost, shorten `duration` — keep 720p (don't drop resolution). Its result is pulled via **SIWX** instead of a paid poll — see Step 6 / endpoints.md.

> **What is SIWX?** "Sign-In-With-X" — an x402 extension where, instead of paying again to fetch your result, you prove you're the wallet that already paid by **signing a short challenge** (like Sign-In-With-Ethereum). The result endpoint answers an unpaid GET with a 402 carrying a `sign-in-with-x` challenge (a nonce); you sign it with the **same wallet that paid** and resend. It's free. Your x402 client must support SIWX to pull these results — many x402 v2 clients do it automatically; if not, you sign the challenge and attach it yourself.

**Sequencing:** one leg at a time; report rows/cost after each; STOP on any error/empty body. Pre-flight the wallet's USDC-on-Base balance (a full run holds several $1 Apify holds transiently).

## Step 0 — Inputs
Capture: **product/service**, **niche**, **region** (countryCode), and target **length** (seconds). Derive niche hashtags + product keywords.
**Then ASK the user which video model before rendering** (costly leg): **Sora 2** (~8–12s, 720p, native audio, ~$0.84, simplest — the default) or **Seedance 2 Pro** (up to 15s, 720p, native audio, ~$3.60 — only if you need 13–15s; needs a **SIWX-capable** client to pull the result). Both bake audio in. Confirm the choice before rendering.

## Workflow

1. **Research (what's winning now).**
   - *Primary (requires Apify API key or a compatible x402 client — see warning above):* `clockworks~tiktok-trends-scraper` (industry + region → trending hashtags/sounds/creators/videos). In parallel: `clockworks~tiktok-hashtag-scraper` (niche hashtags) and `clockworks~tiktok-scraper` (product keywords, sorted most-liked, last week).
   - *Fallback (always works, no API key needed):* 3–4 parallel **Exa searches** via `POST https://stableenrich.dev/api/exa/search` ($0.01 each). Query for: (a) viral TikTok hooks for the niche, (b) pain points and messaging angles for the product, (c) what competitors or adjacent products are saying. Request `contents.summary` in the body to get AI-synthesized takeaways per result. This is proven to work end-to-end and is the recommended starting point.
2. **Engagement.** *Primary:* `clockworks~free-tiktok-scraper` on top candidates. *Fallback:* synthesize engagement signals from Exa summaries — look for view/share counts mentioned in articles, viral hook patterns described in creator guides.
3. **Hooks.** *Primary:* `clockworks~tiktok-transcript-extractor` on top video URLs. *Fallback:* extract spoken hook patterns directly from the Exa research summaries — creator-guide articles typically enumerate the top hook structures with examples.
4. **Reverse-engineer.** Read the winning formats, hooks, and engagement patterns next to the product → write 1–3 new short-form scripts adapting the proven structure.
5. **Reference.** *Primary:* `clockworks~tiktok-video-scraper` with `shouldDownloadVideos`. *Fallback:* skip — proceed to generation with a strong written prompt only.
6. **Generate the short.** Put the spoken line + scene/sound directly in the **prompt** (the model voices it — no separate audio step). The result MP4 already contains audio; download it as soon as the job completes (URLs expire).
   - **Sora 2 (default), ~8–12s:** POST `https://blockrun.ai/api/v1/videos/generations` `{model:"azure/sora-2", prompt, duration_seconds:8}` → it returns a `poll_url`; poll it (GET, x402 same wallet) until `status:"completed"` → download `data[0].url`.
   - **Seedance 2 Pro, up to 15s:** POST `https://stablestudio.dev/api/generate/seedance/t2v` `{prompt, duration:"15", aspectRatio:"9:16", outputResolution:"720p"}` (keep 720p; lower `duration` if you need to cut cost) → it returns a `pollUrl`; **pull it via SIWX** — GET the `pollUrl`, get the `sign-in-with-x` 402 challenge, sign the nonce with the **same wallet that paid**, resend; repeat every ~30–60s (~5–6 min total) until `status:"complete"` → download `result.videoUrl`.

## Output
**ONE finished `.mp4`** — vertical 9:16, **with audio baked in** by the model — plus a short rationale (which winning hooks it was modeled on) and total cost.
