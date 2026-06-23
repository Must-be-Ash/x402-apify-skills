# Local Business Lead Machine — Endpoints & Bodies

x402 paid requests (USDC on Base, `exact`) via any x402 mechanism. Apify actorIds use a **tilde**; run URL: `https://api.apify.com/v2/actors/<actorId>/run-sync-get-dataset-items?maxTotalChargeUsd=0.50` (POST); Apify payTo `0x4aAbE17C239eF71c3A26bA7C2b3e0AeBbfC1DF26`. If a body field is rejected, fetch the live schema (`GET /v2/acts/<actorId>` → build → `.inputSchema`).

## Apify actors

| Actor | Purpose | Body sketch |
|---|---|---|
| `compass~crawler-google-places` | local business discovery | `{ "searchStringsArray": ["dentist"], "locationQuery": "Austin, TX", "maxCrawledPlacesPerSearch": 25 }` |
| `vdrmota~contact-info-scraper` | emails/phones/social URLs from sites | `{ "startUrls": [{"url": "<businessSite>"}], "maxDepth": 1 }` |
| `apify~instagram-profile-scraper` | IG enrichment | `{ "usernames": ["<handle>"] }` |
| `apify~facebook-pages-scraper` | FB page enrichment | `{ "startUrls": [{"url": "<facebookPageUrl>"}] }` |
| `clockworks~tiktok-profile-scraper` | TikTok profile + engagement | `{ "profiles": ["<handle>"], "resultsPerPage": 10 }` |
| `harvestapi~linkedin-company` | company firmographics (no cookies) | `{ "companies": ["<linkedinCompanyUrl or name>"] }` |
| `harvestapi~linkedin-profile-scraper` | person profile + email (no cookies) | `{ "profiles": ["<linkedinProfileUrl>"] }` |

## Supplements (first-party / money-safe gateways)

- **Firecrawl** (Merit): `POST https://stableenrich.dev/api/firecrawl/scrape` · **$0.013 exact** · payTo `0x325bdF6F7efAB24a2210c48c1b64cAb2eAe1d430` · body `{ "url": "<businessSite>", "formats": ["markdown"] }`
- **Exa Search** (first-party): `POST https://api.exa.ai/search` · **$0.007 exact** · payTo `0x6d6E695b09861467c7d462f5AAF31cF3540B9192` · body `{ "query": "<business name> reviews news", "numResults": 5 }`
- **Exa Contents** (first-party): `POST https://api.exa.ai/contents` · **$0.001 exact** · same payTo · body `{ "urls": ["<url>"] }`

## Outreach (IRREVERSIBLE — confirm each send first)

- **Email** — `StableEmail Send` (Merit): `POST https://stableemail.dev/api/send` · **$0.02 exact** · payTo `0xdb5aa553feeb2c3e3d03e8360b36fb0f7e480671` · body `{ "to": "<email>", "subject": "<subj>", "body": "<message>" }` (confirm exact field names via the service's live 402/docs if rejected).
- **Phone** — `AI Phone Call` (Merit): `POST https://stablephone.dev/api/call` · **$0.54 exact** · payTo `0xD219dB8179Bb9C1899eF87f39eebA9D1070c6801` · body `{ "to": "<phone>", "script": "<call script / goal>" }`.

Both require a wallet with sufficient balance and **explicit user confirmation per recipient** before paying.
