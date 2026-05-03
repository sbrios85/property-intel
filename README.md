# Texas Property Intel

Static, browser-based dashboard for tracking motivated seller leads in Nueces County, TX. Designed to deploy on GitHub Pages with zero build step.

## What it does

- **Imports** lead lists (Lis Pendens, Foreclosure, Tax Delinquent, Probate, Code Violations, Liens, Judgments, Absentee Owners) via CSV paste or file upload.
- **Auto-scores** every lead 0–100 based on lead type, distress signals, equity, and address mismatch.
- **Stacks** leads — properties showing up on multiple lists get a STACKED flag and +15 score boost.
- **Filters & sorts** by type, score bucket (Hot ≥70, Warm ≥50, Active ≥30), date, or amount.
- **Comp tool** — enter recent sales, get average $/sqft and ARV.
- **Deal calculator** — MAO with 65/70/75/80% rules.
- **Exports** — Skip Trace CSV (BatchSkipTracing format), GHL CSV (GoHighLevel format), Full Backup (round-trip JSON-equivalent CSV).
- **Persistence** — localStorage, ~5MB limit (~10,000 leads).

## Deploy to GitHub Pages

### Option 1: New repo

```bash
# In a new folder
git init
# Drop index.html in the root
git add index.html README.md
git commit -m "initial commit"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

Then on GitHub:
1. Repo → Settings → Pages
2. Source: **Deploy from a branch**
3. Branch: **main** / folder: **/ (root)**
4. Save. Wait ~1 minute. Your site is live at `https://YOUR_USERNAME.github.io/YOUR_REPO/`

### Option 2: Existing Pages site

Drop `index.html` into your repo root (or any subfolder) and push. If you already have an `index.html`, rename this one to `tpi.html` and access it at `/tpi.html`.

### Option 3: Custom domain

Add a `CNAME` file with your domain (e.g. `propertyintel.example.com`), set the DNS A record to GitHub's Pages IPs, and enforce HTTPS in repo settings.

## File structure

```
.
├── index.html      # Single-file dashboard. Everything in here.
└── README.md       # This file.
```

That's it. No build, no dependencies, no `node_modules`.

## Data model

Each lead is a JSON object:

```js
{
  id: 'i1234567890-1',
  type: 'tax',  // lis_pendens|foreclosure|tax|judgment|lien|probate|code|absentee
  doc: 'TAX',
  doc_number: 'TX-2024-001',
  filed: '2026-01-15',
  owner: 'SMITH JOHN',
  prop_addr: '123 OAK ST',
  prop_city: 'CORPUS CHRISTI',
  prop_zip: '78404',
  mail_addr: 'P.O. BOX 99',
  mail_city: 'DALLAS',
  mail_zip: '75201',
  amount: 5400,        // taxes owed, lien amount, etc.
  est_value: 95000,    // estimated property value (optional)
  score: 78,
  flags: ['STACKED', 'Tax delinquent', 'LLC / corp owner']
}
```

## Scoring formula

Base type score:
- Foreclosure: +35
- Probate: +32
- Tax Delinquent: +30
- Code Violation: +28
- Lis Pendens: +25
- Judgment: +22
- Lien: +18
- Absentee: +15

Bonuses (stack on top):
- STACKED (multi-list match): +15
- High Equity / Free & Clear: +12
- Vacant: +10
- Out-of-state owner: +8
- Tax delinquent flag: +8
- Probate flag: +8
- Auction soon: +8
- Property/Mail address mismatch: +6
- Repeat violation: +6
- Low debt-to-value (<30%): +5
- LLC / Corp owner: +4

Capped at 100. To tune, edit `calcScore()` in `index.html`.

## CSV import format

Header row required. Recognized columns (case-insensitive):

```
owner, prop_addr, prop_city, prop_zip,
mail_addr, mail_city, mail_zip,
amount, est_value, doc_number, filed
```

Extra columns are ignored. Missing columns default to empty/zero.

## Limits

- **No backend.** Static-only. No live scraping, scheduled jobs, or server-side API calls.
- **No cross-device sync.** Data lives in your browser's localStorage on a per-device basis. Use Full Backup before clearing browser data.
- **No skip-trace API.** Export the CSV → upload to your skip-trace provider (BatchSkipTracing, IDI, TLO, etc.) → import results back.
- **No GHL push.** Export CSV → import to GHL.

## Roadmap (if you outgrow static)

To add real automation you'd need a backend. Cheapest path:

1. **Cloudflare Workers** + **Cloudflare D1** (SQLite) — free tier handles thousands of leads, can run scheduled scrapers.
2. **Supabase** — Postgres + auth, 500MB free, REST API.
3. **Backend skip-trace** — call BatchSkipTracing API from your worker, write results back to DB.

The frontend in this repo can talk to any of those — just swap `localStorage` calls for `fetch()` to your API.

## License

MIT or whatever you prefer. Yours to fork.
