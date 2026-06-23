---
name: apify-linkedin-sdr
description: Run a full SDR motion in one agent — find target accounts and decision-makers on LinkedIn, read hiring/company/personal-post buying signals, enrich with outside research, score every prospect with a why-now reason, and run a personalized multi-touch email/phone sequence. Use when the user wants LinkedIn prospecting, account-based targeting, buying-signal research, or an outbound sales sequence. Runs paid x402 services (LinkedIn Apify actors plus Exa plus email/phone), USDC on Base. LinkedIn data is login-gated, so this is for private/catalog use rather than public demos.
---

# Autonomous LinkedIn SDR

Give your product + ICP → the skill finds the right accounts and people, reads their signals, scores each with a "why now", and runs a personalized sequence.

- **Cost:** ~$0.10–0.50 research per prospect batch; + ~$0.02 email / $0.54 phone per touch.
- **Reliability:** LinkedIn data via the no-cookies actors is reliable; **harvestapi is primary, apimaestro is the fallback on every leg.**
- **Status:** LinkedIn is login-gated — private/catalog use, not public launch demos. Public substitute: the `apify-local-business-lead-machine` skill.

## How this skill executes (read first)

Every leg is a **paid x402 call** (USDC on Base, scheme `exact`), made with whatever x402 mechanism the agent has — a **private key + x402 client**, **Coinbase Awal** (Awal CLI), **Agentcash MCP**, **Sponge MCP** (`paid_fetch`), etc. Full URLs/bodies/`payTo`s: **[references/endpoints.md](references/endpoints.md)** — read it first.

**Apify legs:** `POST https://api.apify.com/v2/actors/<actorId>/run-sync-get-dataset-items?maxTotalChargeUsd=0.50` (actorId uses a **tilde**). x402 `exact` = **$1 captured, unused auto-refunded ~1h later**, payTo `0x4aAbE17C239eF71c3A26bA7C2b3e0AeBbfC1DF26`. `maxTotalChargeUsd` floor **$0.50**. **On any leg failure, retry once with the `apimaestro/*` fallback actor (see endpoints.md) before giving up.**

> **⚠️ Apify reliability — READ before any Apify leg (the #1 cause of failed runs):**
> Apify's x402 is **version 2**, and `run-sync-get-dataset-items` holds the connection open while the actor runs — your x402 client's internal timeout (Sponge ~60–90s) is the hard ceiling.
> 1. **Minimize every call** so it finishes in-window: smallest `maxItems` (1–5), one query, no extra add-ons.
> 2. **Heavy/cold-start actors can still exceed the window, and there is NO keyless async fallback** (Apify's async `/runs` start is x402-payable, but fetching the dataset then needs an Apify API key). If an actor keeps timing out, do NOT re-pay it bigger — the timed-out call already placed the $1 hold (refunds ~1h). Either supply an Apify API key or use a lighter alternative.
> 3. **x402-client compatibility:** a **401 "payment payload invalid / could not be verified by the facilitator"** on Apify means your client doesn't interop with Apify's v2 facilitator — not a funds issue. **Sponge `paid_fetch` is verified working with Apify.** Clients like agentcash pay simpler v2 endpoints (Deepgram/Exa/Xona) fine but may 401 on Apify — route the Apify legs through Sponge, keep other legs on your client.

**⚠️ Outward sends (email / phone) are IRREVERSIBLE.** Draft each touch, show it to the user, get **explicit per-send confirmation** before paying. Never auto-send. Operator owns consent/compliance (CAN-SPAM, GDPR, local rules).

**Sequencing:** one leg at a time; report rows/cost after each; STOP on persistent error. Pre-flight the wallet's USDC-on-Base balance.

## Step 0 — Offer + ICP
Capture product/value-prop/ICP (titles, industries, size, geo) + qualifying & buying-signal criteria.

## Workflow

1. **Target.** ICP-driven: `harvestapi~linkedin-company-search` + `harvestapi~linkedin-profile-search`. Account-list: `harvestapi~linkedin-company-employees` on the named companies.
2. **Core data.** Enrich each account with `harvestapi~linkedin-company`, each prospect with `harvestapi~linkedin-profile-scraper` (+ email).
3. **Signals.** Hiring: `curious_coder~linkedin-jobs-scraper`. Company posts: `harvestapi~linkedin-company-posts`. Prospect posts: `harvestapi~linkedin-profile-posts` + topic timing: `harvestapi~linkedin-post-search`.
4. **Research.** `Exa` (first-party, `api.exa.ai`) for company news, funding, product launches beyond LinkedIn.
5. **Score.** Rank by ICP fit + signal strength; attach a "why now" (a recent post, an open role, a funding event).
6. **Sequence (approval-gated).** Draft a personalized multi-touch sequence per prospect — each touch grounded in a real signal (touch 1 email on their post, touch 2 on the hiring signal, touch 3 an AI call). **Confirm each send with the user**, then run via `StableEmail Send` / `AI Phone Call`.

## Output
A prioritized prospect list — per prospect: profile + email, account firmographics, the buying signals found, a fit score + "why now", and the drafted (and optionally sent) sequence. Plus total cost.
