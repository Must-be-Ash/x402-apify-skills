# Real Estate Deal Finder — Endpoints & Bodies

x402 paid requests (USDC on Base, `exact`) via any x402 mechanism. Apify actorIds use a **tilde**; run URL: `https://api.apify.com/v2/actors/<actorId>/run-sync-get-dataset-items?maxTotalChargeUsd=0.50` (POST); Apify payTo `0x4aAbE17C239eF71c3A26bA7C2b3e0AeBbfC1DF26`. Fetch the live schema if a field is rejected.

## Apify actors

| Actor | Purpose | Body sketch |
|---|---|---|
| `tri_angle~real-estate-aggregator` | multi-source sale + rent listings | `{ "location": "Austin, TX", "type": "sale", "maxItems": 50 }` |
| `parseforge~commercial-real-estate-listings-scraper` | commercial (Crexi) variant | `{ "location": "Austin, TX", "maxItems": 25 }` |
| `maxcopell~zillow-detail-scraper` | comps, price history, Zestimate, DOM | `{ "startUrls": [{"url":"<zillowListingUrl>"}] }` |
| `tri_angle~airbnb-scraper` | STR nightly rates / occupancy proxy | `{ "locationQuery": "Austin, TX", "maxItems": 30 }` |
| `voyager~booking-scraper` | STR/hotel rate cross-check | `{ "search": "Austin", "maxItems": 20 }` |
| `compass~google-maps-extractor` | neighborhood amenities + reviews | `{ "searchStringsArray": ["restaurants"], "locationQuery": "<zip/area>", "maxCrawledPlacesPerSearch": 20 }` |
| `afanasenko~zillow-property-agent-data-scraper` | listing-agent contact (for offer) | `{ "startUrls": [{"url":"<zillowListingUrl>"}] }` |

Note: `tri_angle~real-estate-aggregator` exists on Apify but is outside our local snapshot — fetch its live input schema (`GET /v2/acts/tri_angle~real-estate-aggregator`) to confirm field names.

## Supplements

- **Exa Search** (first-party): `POST https://api.exa.ai/search` · **$0.007 exact** · payTo `0x6d6E695b09861467c7d462f5AAF31cF3540B9192` · `{ "query": "<city> real estate market development 2026", "numResults": 5 }`

## Offer send (IRREVERSIBLE — confirm each send first; goes to a real listing agent)

- **Mail** — `PostalForm` (first-party): `POST https://postalform.com/api/machine/orders` · **dynamic $3.40–$200 exact** · payTo `0x95389c432ae7052893d3d8088efc5270b27391f1`. **Call the validate endpoint first** for the exact total (see postalform.com/agents), then pay. Prints & mails a physical letter — highest-stakes send.
- **Email** — `POST https://stableemail.dev/api/send` · **$0.02 exact** · payTo `0xdb5aa553feeb2c3e3d03e8360b36fb0f7e480671` · `{ "to","subject","body" }`
- **Phone** — `POST https://stablephone.dev/api/call` · **$0.54 exact** · payTo `0xD219dB8179Bb9C1899eF87f39eebA9D1070c6801` · `{ "to","script" }`
