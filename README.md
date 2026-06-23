# x402 Apify Skills

A collection of [Claude Agent Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills) that accomplish real business goals by paying for **x402** services per call — [Apify](https://apify.com) actors plus complementary AI endpoints (web search, scraping, image/video generation, voice, email/phone) — settled in **USDC on Base**. No subscriptions, no seats, no API keys: each leg is paid per use over the [x402 protocol](https://x402.org).

Each skill is a self-contained folder with a `SKILL.md` (the workflow) and a `references/endpoints.md` (exact URLs, request bodies, prices, and payment recipients).

## The skills

| Skill | What it does |
|---|---|
| **apify-geo-content-radar** | Measure a brand's visibility in AI answer engines (Google AI Overviews/AI Mode, ChatGPT, Perplexity, Gemini), mine Reddit + X for demand-ranked content ideas → a one-page GEO/content brief. |
| **apify-ugc-content-factory** | Research what's winning on TikTok in a niche, reverse-engineer a script, and render a faceless short with native audio (Sora 2 / Seedance 2 Pro). |
| **apify-local-business-lead-machine** | Find local businesses, enrich each into a scored prospect (contacts, socials, web context), optionally run approval-gated email/phone outreach. |
| **apify-linkedin-sdr** | Full SDR motion: find accounts + decision-makers, read buying signals, score with a "why now," run an approval-gated multi-touch sequence. |
| **apify-real-estate-deal-finder** | From buy-box to offer: scan listings, run comps + income projections, score by strategy, optionally send an offer (mail/email/phone) to the listing agent. |
| **apify-ecommerce-market-intelligence** | Map a category across 80+ marketplaces, compare pricing, mine reviews, rank opportunities, optionally draft a listing with generated creative. |

## How payment works

These skills are **payment-method agnostic**. They specify the HTTP request and the x402 payment facts (scheme, network = Base, asset = USDC, the recipient `payTo`); you bring any x402-capable client to actually pay — e.g. a private key + an x402 client library, Coinbase Awal, an x402-capable MCP, etc.

A few things the skills encode so you don't learn them the hard way:

- **Apify actors** are called via `POST https://api.apify.com/v2/actors/<actorId>/run-sync-get-dataset-items` (actorId uses a `~` tilde). The x402 charge captures up to **$1 and auto-refunds the unused portion ~1h later** — net cost is usually pennies.
- **Keep each Apify call small** (low `maxItems`, one query) — `run-sync` holds the connection open and most x402 clients time out at ~60–90s. There is no keyless async fallback.
- **Apify uses x402 v2.** Some clients interoperate with its facilitator and some don't (a `401 payment payload invalid` means a client/facilitator mismatch, not a funds problem).
- **Some media endpoints use SIWX** ("Sign-In-With-X") to return results: instead of paying again, you prove you're the wallet that paid by signing a short challenge. Your client must support SIWX to pull those results (the relevant skill explains it inline).

## Using a skill

Agent Skills load from `~/.claude/skills/<name>/` (personal) or a project's `.claude/skills/`. To use one:

```bash
# symlink (keeps this repo as the source of truth) or copy
ln -s "$(pwd)/apify-geo-content-radar" ~/.claude/skills/apify-geo-content-radar
```

Then start a new session and invoke it by name (e.g. `/apify-geo-content-radar`) or just describe the task — it triggers on its description. You'll need an x402-capable wallet funded with USDC on Base.

## Safety

- **Outward sends are gated.** Any irreversible action (email, phone call, physical mail) requires explicit per-send confirmation in the skill — nothing is sent automatically.
- **You own compliance.** Outreach is subject to CAN-SPAM, TCPA, GDPR, and local rules; data use is subject to each platform's terms. Use responsibly.
- **Costs are real.** Each leg spends real USDC. Prices and behaviors are documented per endpoint and were verified where noted, but third-party endpoints can change — confirm the live 402 quote before paying.

## Status

Community-built bundles over publicly discoverable x402 services. Endpoints are operated by third parties; this repo is not affiliated with them. Verify pricing and behavior against the live services before relying on a skill in production.
