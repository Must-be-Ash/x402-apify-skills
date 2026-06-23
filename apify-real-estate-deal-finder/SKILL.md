---
name: apify-real-estate-deal-finder
description: From buy-box to a sent offer — scan listing sources, run comps + price-history signals, project long-term and short-term-rental income, read the neighborhood, score each property by return criteria, and (optionally) draft + send an offer to the listing agent by mail/email/phone. Use when the user wants investment-property sourcing, comps/ARV analysis, BRRRR or STR ROI, motivated-seller targeting, or agent outreach. Runs paid x402 services (Apify real-estate/maps actors + Exa + mail/email/phone), USDC on Base.
---

# Real Estate Deal Finder

Give market + budget + strategy (flip / BRRRR / STR) → the skill finds listings, runs comps, projects income, reads the neighborhood, scores deals, and on the ones that pencil, drafts (and with approval, sends) an offer.

- **Cost:** ~$0.20–0.80 research per market scan; + $0.54–$3.40 per offer sent.
- **Reliability:** discovery + comps + neighborhood are rock-solid; STR income is an estimate; offer-send is optional.

## How this skill executes (read first)

Every leg is a **paid x402 call** (USDC on Base, scheme `exact`), made with whatever x402 mechanism the agent has — a **private key + x402 client**, **Coinbase Awal** (Awal CLI), **Agentcash MCP**, **Sponge MCP** (`paid_fetch`), etc. Full URLs/bodies/`payTo`s: **[references/endpoints.md](references/endpoints.md)** — read it first.

**Apify legs:** `POST https://api.apify.com/v2/actors/<actorId>/run-sync-get-dataset-items?maxTotalChargeUsd=0.50` (actorId uses a **tilde**). x402 `exact` = **$1 captured, unused auto-refunded ~1h later**, payTo `0x4aAbE17C239eF71c3A26bA7C2b3e0AeBbfC1DF26`. `maxTotalChargeUsd` floor **$0.50**. Responses are large — extract only needed fields.

> **⚠️ Apify reliability — READ before any Apify leg (the #1 cause of failed runs):**
> Apify's x402 is **version 2**, and `run-sync-get-dataset-items` holds the connection open while the actor runs — your x402 client's internal timeout (Sponge ~60–90s) is the hard ceiling.
> 1. **Minimize every call** so it finishes in-window: smallest `maxItems` (1–5), one query, no extra add-ons.
> 2. **Heavy/cold-start actors can still exceed the window, and there is NO keyless async fallback** (Apify's async `/runs` start is x402-payable, but fetching the dataset then needs an Apify API key). If an actor keeps timing out, do NOT re-pay it bigger — the timed-out call already placed the $1 hold (refunds ~1h). Either supply an Apify API key or use a lighter alternative.
> 3. **x402-client compatibility:** a **401 "payment payload invalid / could not be verified by the facilitator"** on Apify means your client doesn't interop with Apify's v2 facilitator — not a funds issue. **Sponge `paid_fetch` is verified working with Apify.** Clients like agentcash pay simpler v2 endpoints (Deepgram/Exa/Xona) fine but may 401 on Apify — route the Apify legs through Sponge, keep other legs on your client.

**⚠️ Outward sends (mail / email / phone) are IRREVERSIBLE and go to a real listing agent.** Draft the offer, show it + the recipient to the user, get **explicit per-send confirmation** before paying — especially physical **mail** ($3.40+, printed & mailed). Operator owns offer/outreach compliance.

**Sequencing:** one leg at a time; report rows/cost after each; STOP on any error/empty body. Pre-flight the wallet's USDC-on-Base balance.

## Step 0 — Buy-box
Capture market, budget, strategy (flip / BRRRR / STR), and return targets.

## Workflow

1. **Discover.** `tri_angle~real-estate-aggregator` for the location — sale listings (deals) + rent listings (long-term comps), deduped across Zillow/Realtor/Zumper/etc. (Commercial variant: `parseforge~commercial-real-estate-listings-scraper`.)
2. **Comps + signals.** `maxcopell~zillow-detail-scraper` on the shortlist → price history, Zestimate, days-on-market, price cuts → flag under-market + motivated-seller.
3. **Income.** Long-term yield from the aggregator's rent comps; `tri_angle~airbnb-scraper` + `voyager~booking-scraper` for the area → project STR revenue + occupancy.
4. **Neighborhood.** `compass~google-maps-extractor` (+ reviews) → amenities, transit, area sentiment. Optionally `Exa` for local market news/development/schools.
5. **Score.** Rank by strategy: flip margin (ARV − price − rehab est.), cap rate, or cash-on-cash.
6. **Offer (optional, approval-gated).** For top deals, `afanasenko~zillow-property-agent-data-scraper` → listing-agent contact; draft a personalized offer/inquiry; **confirm with the user**, then send via `PostalForm` (mail, $3.40+), `StableEmail Send` (email, ~$0.02), or `AI Phone Call` (phone, $0.54).

## Output
A ranked deal sheet — per property: price + under-market flag, comps, projected long-term & STR income + cap rate, neighborhood read, fit-to-strategy score, listing agent, and (if sent) the drafted offer + send status. STR income is an estimate, not a guarantee. Plus total cost.
