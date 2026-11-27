# INDETAIL Lead Intelligence Collector

A **code-first, scheduled data collector** for INDETAIL Design Studio (Vadodara).
This is not a public website or dashboard. It runs from GitHub Actions (or locally)
and writes CSV reports into `data/`.

## What it collects

| Source | Data collected | Access |
|---|---|---|
| Google Search Console | Search queries, impressions, clicks, CTR, average position, page, country, device | Your verified website property |
| Google Analytics 4 | Sessions, active users, landing pages, source/medium, engagement, key events | Your GA4 property |
| Google Business Profile | Profile performance metrics where supported by your account/API access | Your verified business profile |
| Google Places API | Public business listings relevant to interior design (builders, developers, brokers, architects, commercial businesses) | Google Maps Platform API key; billing may apply |
| Public business websites | Publicly published business phone/email/contact-page candidates | Public pages only; best-effort |
| Instagram Graph API | Authorized professional-account insights and supported hashtag/public-media discovery | Your own eligible Instagram Professional account and Meta permissions |

## What it cannot collect

- Private Google search histories or the identity of people searching for a phrase.
- The identity of anonymous visitors to a website from GA4.
- Private Instagram user data, private profiles, DMs, or hidden contact details.
- A list of people who viewed a competitor's website.
- Guaranteed buyer intent. A public business listing is a prospect to qualify, not a confirmed buyer.

Google Search Console reports queries that caused **your verified property** to appear/click in Google Search; it does not reveal every Google search made by every person. GA4 reports aggregate site activity and events, not visitor identities. See official docs linked below.

## Repository layout

```text
.
├── .github/workflows/daily.yml
├── config.yaml
├── .env.example
├── requirements.txt
├── src/
│   ├── run.py
│   └── connectors/
│       ├── google_search_console.py
│       ├── google_analytics.py
│       ├── google_places.py
│       ├── public_website_contacts.py
│       └── instagram_hashtags.py
└── data/                 # generated CSV files (not committed)
```

## Quick start

1. Create a GitHub repository and upload these files.
2. Create a Google Cloud project. Enable:
   - Google Search Console API
   - Google Analytics Data API
   - Places API (New), if you want prospect discovery
3. Create a Google service account, download its JSON key, and:
   - Add the service-account email as a user on the relevant Search Console property.
   - Add it as Viewer/Analyst on the GA4 property.
4. Add the following GitHub Actions secrets (Settings → Secrets and variables → Actions):
   - `GOOGLE_SERVICE_ACCOUNT_JSON` — the entire service-account JSON content.
   - `GSC_SITE_URL` — e.g. `https://indetail-design.com/` (must match the verified property).
   - `GA4_PROPERTY_ID` — numeric GA4 property ID.
   - `GOOGLE_MAPS_API_KEY` — optional, for Places lead discovery.
   - `META_ACCESS_TOKEN`, `META_IG_USER_ID` — optional, for Instagram features.
5. Edit `config.yaml` with target city and prospect search phrases.
6. Run the workflow manually once from Actions, then it runs daily.

Local run:
```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
# Save service account JSON as credentials/google-service-account.json
export GOOGLE_APPLICATION_CREDENTIALS=credentials/google-service-account.json
python -m src.run
```

## Outputs

- `data/gsc_queries.csv`
- `data/ga4_landing_pages.csv`
- `data/ga4_traffic_sources.csv`
- `data/places_prospects.csv`
- `data/public_contact_candidates.csv`
- `data/instagram_hashtag_research.csv` (only if Meta credentials and permissions are available)

GitHub Actions uploads these as a downloadable workflow artifact. To retain reports between runs,
enable artifact retention or add a private storage destination.

## Privacy and responsible use

Use only public business contact information and data from accounts you administer or are
authorised to access. Respect website terms, robots directives, API quotas, and opt-outs.
Do not collect sensitive personal data or build lists of private individuals. Review and verify
every contact before outreach. The public contact parser is best-effort and does not bypass
logins, paywalls, CAPTCHAs, or access controls.

## Official documentation

- Search Console Search Analytics API: https://developers.google.com/webmaster-tools/v1/searchanalytics
- Search Console query examples: https://developers.google.com/webmaster-tools/v1/how-tos/search_analytics
- Google Analytics Data API: https://developers.google.com/analytics/devguides/reporting/data/v1
- Google Places API (New): https://developers.google.com/maps/documentation/places/web-service/text-search
- Instagram Graph API: https://developers.facebook.com/docs/instagram-platform/instagram-graph-api
- Instagram hashtag search: https://developers.facebook.com/docs/instagram-platform/instagram-graph-api/hashtag-search
