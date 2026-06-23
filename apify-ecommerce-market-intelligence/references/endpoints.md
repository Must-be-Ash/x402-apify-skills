# E-commerce Market Intelligence — Endpoints & Bodies

x402 paid requests (USDC on Base, `exact`) via any x402 mechanism. Apify actorIds use a **tilde**; run URL: `https://api.apify.com/v2/actors/<actorId>/run-sync-get-dataset-items?maxTotalChargeUsd=0.50` (POST); Apify payTo `0x4aAbE17C239eF71c3A26bA7C2b3e0AeBbfC1DF26`. Fetch the live schema if a field is rejected.

## Apify actor

`apify~e-commerce-scraping-tool` — one actor, multiple modes (search / listing / product / reviews / sellers) across 80+ marketplaces + Google Shopping.
```json
{ "mode": "search", "query": "stainless steel water bottle", "marketplaces": ["amazon","ebay","googleShopping"], "maxItems": 30 }
```
Then per-product: `{ "mode": "product", "urls": ["<productUrl>"] }` and reviews: `{ "mode": "reviews", "urls": ["<productUrl>"], "sort": "lowest", "maxItems": 50 }`. Field/mode names vary — confirm via the live input schema (`GET /v2/acts/apify~e-commerce-scraping-tool`). Leave any built-in AI-summary add-on OFF.

## Supplements

- **Exa Search** (first-party): `POST https://api.exa.ai/search` · **$0.007 exact** · payTo `0x6d6E695b09861467c7d462f5AAF31cF3540B9192` · `{ "query": "<category> demand trend", "numResults": 5 }`
- **Exa Contents** (first-party): `POST https://api.exa.ai/contents` · **$0.001 exact** · same payTo · `{ "urls": ["<url>"] }`

## Listing creative — DEFAULT: Orthogonal Nano Banana 2

`POST https://x402.orth.sh/nano-banana-2/v1beta/models/gemini-3.1-flash-image-preview:generateContent` · **$0.05 exact** (Base) · payTo `0x8bF5C401A41ebe508E259f62612B0E22c3D43B75` · tier-2 trusted. Gemini-style body:
```json
{ "contents": [{ "parts": [{ "text": "<product/listing image prompt>" }] }], "generationConfig": { "responseModalities": ["IMAGE"] } }
```

**Alternatives (all `exact`):**
- Orthogonal Z.ai CogView ($0.01, cheaper): `POST https://x402.orth.sh/zai/api/paas/v4/images/generations`
- Orthogonal Nano Banana Pro ($0.15, higher quality): `POST https://x402.orth.sh/nano-banana/v1beta/models/gemini-3-pro-image-preview:generateContent`
- Xona FLUX.2 Pro ($0.05): `POST https://api.xona-agent.com/base-main/image/flux-2-pro` (payTo via live 402)
- BlockRun ($0.04 fixed/pull): `POST https://blockrun.ai/api/v1/images/generations` `{model:"openai/gpt-image-2", prompt, size, n}` (returns 400 until a valid body, then 402)
- stablestudio.dev/Merit (dynamic-max): `POST https://stablestudio.dev/api/generate/gpt-image-2/generate` — 402 authorizes a **$10 max** but settles dynamically + refunds ~1h, net ≈ display. Skip fal.ai (broken).

## Supplier outreach (IRREVERSIBLE — confirm send first)

- **Email** — `POST https://stableemail.dev/api/send` · **$0.02 exact** · payTo `0xdb5aa553feeb2c3e3d03e8360b36fb0f7e480671` · `{ "to","subject","body" }`
