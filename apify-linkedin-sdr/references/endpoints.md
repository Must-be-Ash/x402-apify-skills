# LinkedIn SDR — Endpoints & Bodies

x402 paid requests (USDC on Base, `exact`) via any x402 mechanism. Apify actorIds use a **tilde**; run URL: `https://api.apify.com/v2/actors/<actorId>/run-sync-get-dataset-items?maxTotalChargeUsd=0.50` (POST); Apify payTo `0x4aAbE17C239eF71c3A26bA7C2b3e0AeBbfC1DF26`. Fetch the live schema if a body field is rejected. **harvestapi is primary; on failure retry the apimaestro fallback** (verify the fallback actorId live if unsure).

## LinkedIn actors (primary → fallback)

| Stage | Primary (`harvestapi~…`) | Fallback (`apimaestro~…`) | Body sketch |
|---|---|---|---|
| Find accounts | `linkedin-company-search` | `linkedin-companies-search-scraper` | `{ "search": "<industry/keywords>", "maxItems": 25 }` |
| Find people | `linkedin-profile-search` | `linkedin-profile-search-scraper` | `{ "search": "<title> <industry>", "locations": ["<geo>"], "maxItems": 25 }` |
| Buying committee | `linkedin-company-employees` | `linkedin-company-employees-scraper-no-cookies` | `{ "companies": ["<companyUrl>"], "maxItems": 25 }` |
| Firmographics | `linkedin-company` | `linkedin-company-detail` | `{ "companies": ["<companyUrl or name>"] }` |
| Profile + email | `linkedin-profile-scraper` | `linkedin-profile-detail` (+ `-batch`) | `{ "profiles": ["<profileUrl>"] }` |
| Hiring signal | `curious_coder~linkedin-jobs-scraper` | `apimaestro~linkedin-jobs-scraper-api` | `{ "companyName": "<company>", "rows": 25 }` |
| Company posts | `linkedin-company-posts` | `linkedin-company-posts` | `{ "companies": ["<companyUrl>"], "maxItems": 15 }` |
| Prospect posts | `linkedin-profile-posts` | `linkedin-profile-posts` | `{ "profiles": ["<profileUrl>"], "maxItems": 15 }` |
| Post/topic search | `linkedin-post-search` | `linkedin-posts-search-scraper-no-cookies` | `{ "search": "<topic/company>", "maxItems": 20 }` |

(Field names vary per actor — confirm via the live input schema if a call 400s.)

## Supplements

- **Exa Search** (first-party): `POST https://api.exa.ai/search` · **$0.007 exact** · payTo `0x6d6E695b09861467c7d462f5AAF31cF3540B9192` · body `{ "query": "<company> funding news", "numResults": 5, "category": "news" }`

## Outreach (IRREVERSIBLE — confirm each send first)

- **Email** — `POST https://stableemail.dev/api/send` · **$0.02 exact** · payTo `0xdb5aa553feeb2c3e3d03e8360b36fb0f7e480671` · `{ "to","subject","body" }`
- **Phone** — `POST https://stablephone.dev/api/call` · **$0.54 exact** · payTo `0xD219dB8179Bb9C1899eF87f39eebA9D1070c6801` · `{ "to","script" }`
