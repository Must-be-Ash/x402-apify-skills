---
name: apify-ecommerce-market-intelligence
description: Map a product category across 80+ marketplaces, compare pricing, mine reviews for unmet needs, validate demand, and rank product/pricing opportunities — then optionally draft the listing with generated creative or email a supplier. Use when the user wants e-commerce/marketplace research, competitive pricing analysis, product-gap or arbitrage discovery, review mining, or listing creation. Runs paid x402 services (Apify e-commerce actor + Exa + image-gen + email), USDC on Base.
---

# E-commerce Market Intelligence

Name a category + goal → the skill maps the category across marketplaces, compares pricing, mines reviews for unmet needs, validates demand, and ranks opportunities — optionally drafting a listing with creative or emailing a supplier.

- **Cost:** ~$0.20–1.00 research per category scan; + ~$0.04/image and ~$0.02/email if used.
- **Reliability:** the e-commerce tool covers discovery/price/reviews/sellers; demand-validation + build steps are optional.

## How this skill executes (read first)

Every leg is a **paid x402 call** (USDC on Base, scheme `exact`), made with whatever x402 mechanism the agent has — a **private key + x402 client**, **Coinbase Awal** (Awal CLI), **Agentcash MCP**, **Sponge MCP** (`paid_fetch`), etc. Full URLs/bodies/`payTo`s: **[references/endpoints.md](references/endpoints.md)** — read it first.

**Apify legs:** `POST https://api.apify.com/v2/actors/<actorId>/run-sync-get-dataset-items?maxTotalChargeUsd=0.50` (actorId uses a **tilde**). x402 `exact` = **$1 captured, unused auto-refunded ~1h later**, payTo `0x4aAbE17C239eF71c3A26bA7C2b3e0AeBbfC1DF26`. `maxTotalChargeUsd` floor **$0.50**. Responses are large — extract only needed fields.

> **⚠️ Apify reliability — READ before any Apify leg (the #1 cause of failed runs):**
> Apify's x402 is **version 2**, and `run-sync-get-dataset-items` holds the connection open while the actor runs — your x402 client's internal timeout (Sponge ~60–90s) is the hard ceiling.
> 1. **Minimize every call** so it finishes in-window: smallest `maxItems` (1–5), one query, no extra add-ons.
> 2. **Heavy/cold-start actors can still exceed the window, and there is NO keyless async fallback** (Apify's async `/runs` start is x402-payable, but fetching the dataset then needs an Apify API key). If an actor keeps timing out, do NOT re-pay it bigger — the timed-out call already placed the $1 hold (refunds ~1h). Either supply an Apify API key or use a lighter alternative.
> 3. **x402-client compatibility:** a **401 "payment payload invalid / could not be verified by the facilitator"** on Apify means your client doesn't interop with Apify's v2 facilitator — not a funds issue. **Sponge `paid_fetch` is verified working with Apify.** Clients like agentcash pay simpler v2 endpoints (Deepgram/Exa/Xona) fine but may 401 on Apify — route the Apify legs through Sponge, keep other legs on your client.

**Image-gen:** default to **Orthogonal — Nano Banana 2** (`x402.orth.sh`, **$0.05 `exact`**, tier-2 trusted). Alternatives if preferred: Orthogonal Z.ai CogView ($0.01, cheaper/lower quality) or Nano Banana Pro ($0.15, higher quality); Xona FLUX.2 Pro ($0.05); BlockRun fixed ~$0.04/pull; or stablestudio.dev (Merit) which authorizes a $10 max but settles dynamically + refunds ~1h (net ≈ display). Skip fal.ai (broken via our gateways).
**⚠️ Email is IRREVERSIBLE** — draft it, show it to the user, confirm before sending. Operator owns sourcing-outreach compliance.

**Sequencing:** one leg at a time; report rows/cost after each; STOP on any error/empty body. Pre-flight the wallet's USDC-on-Base balance.

## Step 0 — Intent
Capture the goal (what to sell / how to price / what to build), the category, and target marketplaces.

## Workflow

1. **Map + price + reviews.** `apify~e-commerce-scraping-tool` (one actor, multiple modes): keyword search across the chosen marketplaces + Google Shopping → category landscape + price distribution; product + seller details for top products; reviews (sorted lowest-rated) → the pain-point corpus. (Leave the actor's built-in AI-summary OFF — you do the synthesis.)
2. **Validate demand.** `Exa` (first-party, `api.exa.ai`) `/search` + `/contents` → real demand/trends + brand context beyond the marketplace.
3. **Opportunities.** Synthesize a ranked list: product gaps, pricing/arbitrage plays, review-driven unmet needs — each with a rationale.
4. **Build (optional).** For a chosen opportunity, draft listing copy and generate product/listing creative via **Orthogonal Nano Banana 2** (~$0.05; see endpoints.md for alternatives).
5. **Source (optional, approval-gated).** Draft a supplier/brand outreach email; **confirm with the user**, then send via `StableEmail Send` (~$0.02).

## Output
A category intelligence report — cross-platform landscape + price map, top complaint/unmet-need themes, a demand read, a ranked opportunity list, and (optionally) a drafted listing with creative or a sent sourcing email. Plus total cost.
