---
name: apify-local-business-lead-machine
description: Find local businesses, enrich each into a scored prospect with a context brief (contacts, socials, website/web context), and optionally run personalized email/phone outreach. Use when the user wants local lead generation, a prospect list for a category + area, business contact enrichment, or local outreach. Runs paid x402 services (Apify discovery/contacts/social actors + Firecrawl + Exa + email/phone), USDC on Base.
---

# Local Business Lead Machine

Tell the agent who you sell to and why → it finds matching local businesses, pulls contacts + social profiles, reads their site and the web for context, scores each lead, and (optionally, with approval) sends a personalized message.

- **Cost:** ~$0.10–0.40 research per batch; + ~$0.02 email / $0.54 phone per contact if outreach runs.
- **Reliability:** discovery + enrichment core is rock-solid; social legs run only when a profile URL exists.

## How this skill executes (read first)

Every leg is a **paid x402 call** (USDC on Base, scheme `exact`), made with whatever x402 mechanism the agent has — a **private key + x402 client**, **Coinbase Awal** (Awal CLI), **Agentcash MCP**, **Sponge MCP** (`paid_fetch`), etc. Full URLs/bodies/`payTo`s: **[references/endpoints.md](references/endpoints.md)** — read it first.

**Apify legs:** `POST https://api.apify.com/v2/actors/<actorId>/run-sync-get-dataset-items?maxTotalChargeUsd=0.50` (actorId uses a **tilde**). x402 `exact` = **$1 captured, unused auto-refunded ~1h later**, payTo `0x4aAbE17C239eF71c3A26bA7C2b3e0AeBbfC1DF26`. `maxTotalChargeUsd` floor **$0.50**. Responses can be large — extract only needed fields.

> **⚠️ Apify reliability — READ before any Apify leg (the #1 cause of failed runs):**
> Apify's x402 is **version 2**, and `run-sync-get-dataset-items` holds the connection open while the actor runs — your x402 client's internal timeout (~60–90s) is the hard ceiling.
> 1. **Minimize every call** so it finishes in-window: smallest `maxItems` (1–5), one query, no extra add-ons.
> 2. **Heavy/cold-start actors can still exceed the window, and there is NO keyless async fallback** (Apify's async `/runs` start is x402-payable, but fetching the dataset then needs an Apify API key). If an actor keeps timing out, do NOT re-pay it bigger — the timed-out call already placed the $1 hold (refunds ~1h). Either supply an Apify API key or use a lighter alternative.
> 3. **x402-client compatibility:** a **401 "payment payload invalid / could not be verified by the facilitator"** on Apify means your client doesn't interoperate with Apify's v2 facilitator — not a funds issue. Use a client that speaks x402 v2; some clients pay simpler v2 endpoints fine but 401 on Apify — if that happens, switch clients for the Apify legs.

**⚠️ Outward sends (email / phone) are IRREVERSIBLE.** Draft the message, show it to the user, and get **explicit per-send confirmation** before paying. Never auto-send. The operator owns consent/compliance (CAN-SPAM, TCPA, local rules).

**Sequencing:** one leg at a time; report rows/cost after each; STOP on any error/empty body. Pre-flight the wallet's USDC-on-Base balance.

## Step 0 — Intent
Capture: who they sell to, the offer, qualifying criteria, and the outreach goal (drives scoring + messaging).

## Workflow

1. **Discover.** `compass~crawler-google-places` for the category + area → business list (name, site, rating, hours, base contact).
2. **Contacts.** `vdrmota~contact-info-scraper` on each business site → emails, phones, social profile URLs.
3. **Social enrichment.** Route each found social URL to its actor (skip platforms with no URL): Instagram → `apify~instagram-profile-scraper`; Facebook → `apify~facebook-pages-scraper`; TikTok → `clockworks~tiktok-profile-scraper`; LinkedIn company → `harvestapi~linkedin-company`, person → `harvestapi~linkedin-profile-scraper`.
4. **Web context.** `Firecrawl` (Merit, `stableenrich.dev`) on the business's own site → LLM-ready text; `Exa` (first-party, `api.exa.ai`) `/search` + `/contents` for news/mentions. Build a per-business context profile.
5. **Score.** Rank each lead against the criteria (fit, signal, reachability) with a rationale.
6. **Outreach (optional, approval-gated).** For top leads, draft a personalized message grounded in the context profile. **Show each draft + recipient to the user and get explicit confirmation**, then send via `StableEmail Send` (email, ~$0.02) or `AI Phone Call` (phone, $0.54).

## Output
A ranked, scored prospect list — each with contacts, social presence, a website/web context brief, a fit score + rationale, and (if outreach ran) the drafted message + send status. Plus total cost.
