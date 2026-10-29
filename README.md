# How to Scrape Job Postings Across Europe: 12 Boards, 7 Countries, One Schema

> **Run it now, no code: [europe-jobs-scraper on the Apify Store](https://apify.com/bibian/europe-jobs-scraper)** &nbsp;|&nbsp; $2.00 per 1,000 postings &nbsp;|&nbsp; no browser for the default sources

Buying and maintaining a separate scraper per country is slow. This Actor reads 12 major European job boards — [Free-Work](https://www.free-work.com), [France Travail](https://candidat.francetravail.fr), [HelloWork](https://www.hellowork.com), [Reed](https://www.reed.co.uk), [Arbeitsagentur](https://www.arbeitsagentur.de), [Stellenanzeigen](https://www.stellenanzeigen.de), [Jobware](https://www.jobware.de), [Infoempleo](https://www.infoempleo.com), [Trabajos](https://www.trabajos.com), [TicJob](https://www.ticjob.es), [Arbetsformedlingen](https://arbetsformedlingen.se) and [Jobs.ch](https://www.jobs.ch) — plus the official EU portal [EURES](https://europa.eu/eures), in a single run, deduplicated and mapped to one unified schema: France, Germany, Spain, Great Britain, Sweden, Switzerland and Poland.

Searches this page answers: european jobs scraper, multi country job board api, germany france spain jobs scraper, eu job postings dataset.

## What you can do with it

- **Search job postings across 7 European countries in one run**, instead of buying and maintaining a dozen separate country scrapers.
- **Filter once, everywhere**: `keywords`, `location`, `remote` and `publishedWithinDays` apply the same way across every source.
- **Compare pay and location across countries**: every record shares the same `salaryMin`/`salaryMax`, `city`, `country` and `employmentType` fields, whichever site or country it came from.
- **Cover smaller markets with EURES**: the official EU job portal fills in France, Germany, Spain, Sweden, Switzerland and Poland alongside the national boards, deduplicated against them.
- **Monitor new postings daily**: schedule it with `onlyNew` checked and `publishedWithinDays: 1` to get, and pay for, only what is new since the last run.

A real sample (five results across five different sources, personal data removed) is in [`sample-output.json`](sample-output.json).

## Output fields

| Field | Type | Meaning |
|---|---|---|
| `id` | string | Stable identifier (`<source>:<native id>`), used to deduplicate between runs. |
| `url` | string | Public page of the posting. |
| `title` | string | Job title as published. |
| `companyName` | string or null | Employer name; `null` when hidden (then `confidentialCompany: true`). |
| `locationLabel`, `city`, `region`, `postalCode`, `country` | — | Where the job is based; `country` is always an ISO 3166-1 alpha-2 code (`FR`, `DE`, `ES`, `GB`, `SE`, `CH`, `PL`). |
| `remote` | string | `remote`, `hybrid`, `onsite` or `unknown`. |
| `employmentType` | string | `full_time`, `part_time`, `contract`, `freelance`, `internship`, `apprenticeship`, `temporary` or `unknown`. |
| `salaryMin` / `salaryMax` / `salaryCurrency` / `salaryPeriod` | — | Salary range as numbers, ISO 4217 currency, and `hour`/`day`/`month`/`year`. |
| `datePosted`, `validThrough` | string or null | Publication and expiry date, ISO 8601. |
| `description` | string or null | Plain-text description, only when `includeDescriptions` is on. No personal data. |
| `skills` | string[] | Skills listed by the site, if any. |
| `source`, `sourceId`, `scrapedAt` | — | Host name of the source site, its native id, and when this Actor read it. |

Site-specific fields are kept in addition to these. EURES also carries `occupationCodes` (raw ESCO URIs) and `locationCountries` (every country for a cross-border vacancy).

### Default sources vs. opt-in sources

The 13 default sources (Free-Work, France Travail, HelloWork, Reed, Arbeitsagentur, Stellenanzeigen, Jobware, Infoempleo, Trabajos, TicJob, Arbetsformedlingen, Jobs.ch, EURES) answer directly, no proxy or browser needed, so a default run stays fast and cheap. Five more boards exist in `sources` but are off by default because they need a residential proxy or heavier load: **Welcome to the Jungle** (FR), **StepStone** (DE), **MeineStadt** (DE), **Empleate** (ES) and **Pracuj** (PL, the only source for Poland — add it explicitly to get Polish postings).

## Pricing

Pay per result: **$2.00 per 1,000 postings**, plus a small start fee. You only pay for the postings you receive, deduplicated across every active source before being counted or charged. Store discounts apply automatically on paid Apify plans.

## Run it from your code

JavaScript:

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });
const run = await client.actor('bibian/europe-jobs-scraper').call({
  countries: ['FR', 'DE', 'ES', 'GB'],
  keywords: 'developer',
  maxItems: 50,
});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
```

Python:

```python
from apify_client import ApifyClient

client = ApifyClient(token="<APIFY_TOKEN>")
run = client.actor("bibian/europe-jobs-scraper").call(run_input={
    "countries": ["FR", "DE", "ES", "GB"],
    "keywords": "developer",
    "maxItems": 50
})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

The complete files are [`examples/run.js`](examples/run.js) and [`examples/run.py`](examples/run.py). Never put your token in the code: read it from the `APIFY_TOKEN` environment variable as above.

## Monitor new postings daily and connect your tools

1. **Schedule it.** Save a task, pick your countries and keywords, and schedule it daily with Apify Schedules. Tick **Only new results** and each run returns, and charges, only the postings earlier runs of the same search did not deliver.
2. **Push the results where you work.** Add an Apify integration or a webhook to the task: Google Sheets, Slack, Make, Zapier, n8n or your own endpoint.
3. **Call it from your stack.** The API and the JavaScript and Python clients return the dataset as JSON, CSV or Excel.
4. **Let an AI agent use it.** Through the [Apify MCP server](https://docs.apify.com/platform/integrations/mcp), assistants such as Claude or ChatGPT can search the Store, read this page and run the actor as a tool.

## Why this one

12 national boards plus the official EU portal, 7 countries, one schema. `country` and `source` on every row to filter or split. Deduplicated across every source, by URL and by (title, company, city). No connection, no login; the default sources need no browser or residential proxy either.

## Good to know

- `employmentType` and `remote` are read from each site's own fields when it has one; when a source carries no such field, they are inferred from the title and description, and left `unknown` rather than guessed when nothing matches. EURES exposes no teleworking signal at all: its `remote` is always `unknown`.
- EURES has no UK coverage: requesting `GB` only engages Reed.
- No personal data: contact details of individuals are stripped from descriptions; only companies and public employers are named.
- This is an unofficial tool, not affiliated with any of the job boards listed above. Check each site's terms and the data protection rules that apply to your use.

## Frequently asked questions

**Do I need a browser or a proxy?** No, not for the 13 default sources. Five opt-in sources (Welcome to the Jungle, StepStone, MeineStadt, Empleate, Pracuj) need a residential proxy; turning them on in `sources` costs more per run but not per result.

**Which countries are covered?** France, Germany, Spain, Great Britain, Sweden, Switzerland and Poland. Pick any mix in `countries`.

**Can I get only new postings each day?** Yes: set `publishedWithinDays` to `1`, tick `onlyNew`, and schedule the Actor daily.

**Is it legal to scrape?** The actor collects publicly visible postings and follows each site's robots.txt rules. How you use the data is your responsibility: check the sites' terms and the laws that apply to you, for example the GDPR.

**Something does not work?** Open an issue in the Issues tab. Every public actor is test-run daily and the broken ones are repaired.

## More job scrapers by the same author

- [EURES Jobs Scraper (~30 EU/EEA countries)](https://apify.com/bibian/eures-jobs-scraper)
- [Free-Work Jobs & Freelance Missions Scraper (France)](https://apify.com/bibian/free-work-jobs-scraper)
- [Infoempleo Jobs Scraper (Spain)](https://apify.com/bibian/infoempleo-jobs-scraper)
- [Arbeitsagentur Jobs Scraper (Germany)](https://apify.com/bibian/arbeitsagentur-jobs-scraper)
- [Remote Jobs Worldwide Scraper (8 boards)](https://apify.com/bibian/remote-jobs-worldwide-scraper)

## Try it

**[Open europe-jobs-scraper on the Apify Store](https://apify.com/bibian/europe-jobs-scraper)**

The code in `examples/` is released under the MIT license (see [LICENSE](LICENSE)). The data belongs to its original publishers.
