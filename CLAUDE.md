# FM Portfolio Home Base

Interactive property/portfolio map web app for **Fannie May**. Static site (no build
step) served from `docs/` via GitHub Pages. Data comes from Smartsheet and is
refreshed daily by a GitHub Action.

- Repo: `skypep737-sys/FM-Portfolio-Home-Base` (branch `main`)
- Live site: GitHub Pages, built from the `/docs` folder on `main`
- This is the Fannie May deployment of a reusable template (see "Related projects")

## Run it locally

It's plain HTML/CSS/JS — just serve the `docs/` folder:

```
python3 -m http.server 4599 --directory docs
```

Then open http://localhost:4599. There is no install/build/compile step.

## Project layout

- `docs/index.html` — the entire app (HTML + CSS + JS inline). This is the main file
  you edit for any UI change. ~1900 lines, organized in sections.
- `docs/client.js` — `CLIENT_CONFIG`: branding (appName, brandLetter, primaryColor),
  `dealTypes` (must match the Smartsheet "Deal Type" values + their pin colors),
  and the default map center/zoom. Edit this to re-skin for a client.
- `client-config.json` — maps the app's internal field names to the exact Smartsheet
  column titles. `column_map` is for the main properties sheet; `survey_column_map`
  is for survey sheets. Keys (left) are fixed; values (right) match your sheet.
- `docs/properties.json` / `docs/surveys.json` — generated data (do NOT hand-edit;
  the daily refresh overwrites them).
- `fetch_and_geocode.py` — pulls from Smartsheet, geocodes addresses, writes the two
  JSON files. Run by the GitHub Action.
- `geocode_cache.json` — cached lat/lng so we don't re-geocode unchanged addresses.
- `docs/leaflet.css` / `docs/leaflet.js` — self-hosted Leaflet (the map library).
- `.github/workflows/refresh.yml` — "Daily data refresh" action.

## The app's three views

- **Map View** — Leaflet map of portfolio locations; clicking a pin opens a popup and
  highlights it in the sidebar list. "View more" opens the deal detail modal.
- **Deal View** — sortable table (Site ID, Store Name, Deal Type, Deal Status, Latest
  Notes, Lease Expiration) with a per-row "Details" button opening a full site modal.
- **Survey View** — table of survey/prospect sites + a map below it showing only survey
  sites, with a toggle to overlay portfolio locations.

## Data refresh

- `.github/workflows/refresh.yml` runs `fetch_and_geocode.py` daily at 08:00 UTC and on
  manual dispatch (`gh workflow run "Daily data refresh"` or the GitHub UI).
- It reads these **GitHub repo Secrets** (already set; not stored locally):
  - `SMARTSHEET` — Smartsheet API token
  - `SMARTSHEET_SHEET_ID` — the main properties sheet ID
  - `SURVEY_FOLDER_ID` — Smartsheet folder containing the survey sheets
- The action commits regenerated `properties.json` / `surveys.json` back to `main`.

## Conventions / gotchas

- All UI lives in `docs/index.html`. Pin colors and the sqft/base_rent column mapping
  are intentionally per-client.
- Smartsheet sometimes returns numeric Site IDs as `"280.0"` — the UI strips the
  trailing `.0` for display via the `fmtId()` helper. Keep raw values in data attributes.
- After editing `docs/index.html`, verify in a browser (serve `docs/` and click through
  Map / Deal / Survey views) before committing.
- GitHub Pages serves from `/docs` on `main`, so a push to `main` updates the live site.

## Related projects (same template, other clients)

- `~/property-map` — Salvation Army deployment. NOTE its differences: it uses CDN-hosted
  Leaflet (not self-hosted), a single Esri tile layer, and client labels like "Drumline".
  When propagating shared UI changes there, apply surgical edits — do not wholesale-copy
  `index.html`, or you'll break those specifics.
- `FM-mapping` — the generic template repo; its `index.html` can usually be copied 1:1
  from this repo.
