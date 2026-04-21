# GriffinSIGHT Ingest

Browser-based curator for building labeled image datasets, designed for computer vision
training workflows (Automatic Target Recognition, object detection, classification).

Single self-contained HTML file. No build step, no server, no backend.
Runs locally via `file://` or hosted behind any static HTTP server.

---

## What it does

1. Queries multiple image APIs in parallel from the browser
2. Presents candidates in a reviewable grid — KEEP or DROP each tile
3. Exports the approved set as a `.zip` with the images plus `manifest.csv`
   (license, source URL, artist, credit, caption, dimensions) for training-data provenance

## Sources

| Source | Status | Auth | Notes |
|---|---|---|---|
| Wikimedia Commons | Live | None | PD / CC · universal CORS |
| Library of Congress | Live | None | US gov · historical · PD |
| Openverse | Live | None | CC aggregator across Flickr, museums, Wikimedia |
| DVIDS | Live | API key (origin-bound) | DoD imagery · aerial / operational |
| NARA Catalog | Live | API key | US National Archives · PD · 10k queries/month |
| *(user-added)* | Live | Varies | JSON-configured via "+ ADD SOURCE" |

API keys live in the browser's `localStorage` only — never committed to the repo,
never logged, never shipped with the HTML.

## Obtaining API keys

### DVIDS

1. Register at <https://api.dvidshub.net>
2. Create an application and register the origin this page is served from
   (scheme + host, no path — e.g. `https://gabe-334.github.io`)
3. Receive a key of the form `key-xxxxxxxxxxxxx`
4. In the tool, tap the DVIDS card's gear icon and paste the key

DVIDS binds keys to origins. If you move this tool to a different host, re-register.

### NARA Catalog

1. Email `Catalog_API@nara.gov` with your name and email, requesting a **read-only API key**
2. Receive a key by reply email (typically 1–2 business days)
3. In the tool, tap the NARA card's gear icon and paste the key

Rate limit: 10,000 queries/month per key.

## Deployment

### GitHub Pages

1. Upload `index.html` + `README.md` to the repo root
2. Settings → Pages → **Deploy from a branch** · branch `main` · folder `/ (root)` → **Save**
3. Wait ~60 seconds; a green banner appears with the live URL
4. Register that URL (scheme + host only) with DVIDS if using it

### Local

Open `index.html` directly in any modern browser. Commons, LoC, and Openverse work from
the `file://` origin. DVIDS and NARA keyed calls require a hosted URL registered with
each service.

### Other static hosts

Any platform that serves a static HTML file works: Cloudflare Pages, Netlify, nginx,
plain Apache, even `python -m http.server`. The tool has zero server-side dependencies.

## Workflow

1. Pick a class from the presets row or type a custom query
2. Sources panel (left) shows all sources with live hit counts
3. Tap a source card to include/exclude it from this session
4. Tap the gear on DVIDS / NARA to paste an API key
5. Tap **▶ GO** — all included sources fire in parallel
6. On each tile: **✓ KEEP** moves to cart · **✗ DROP** discards
7. **EXPORT .ZIP** downloads images + manifest

## Custom sources

Add any REST API that returns JSON with image URLs via **+ ADD SOURCE**.
Config format supports URL templates, auth headers, offset/page pagination,
and field mapping with dot-path lookups.

Example:

```json
{
  "name": "Pexels",
  "endpoint": "https://api.pexels.com/v1/search",
  "headers": { "Authorization": "YOUR_KEY" },
  "params": { "query": "{query}", "per_page": "{limit}", "page": "{page}" },
  "pagination": { "strategy": "page", "startsAt": 1 },
  "response": { "resultsPath": "photos" },
  "map": {
    "id": "id", "title": "alt", "pageUrl": "url",
    "thumb": "src.medium", "full": "src.original",
    "artist": "photographer", "license": "=Pexels License"
  }
}
```

Placeholders: `{query}` `{limit}` `{offset}` `{page}`.
Literal values in `map` start with `=`.

## Export format

```
griffinsight_<class>_<timestamp>.zip
└── <class>/
    ├── 0001_<source>_<title>.jpg
    ├── 0002_<source>_<title>.jpg
    ├── ...
    ├── manifest.csv     ← full provenance per image
    └── README.txt       ← class, query, per-source counts
```

`manifest.csv` columns:

```
filename, source, source_url, page_url, license, artist, credit,
date, width, height, bytes, mime, caption
```

Individual image fetch failures don't abort the export — failed rows are tagged
`FAILED_<source>_<id>` in the manifest with the HTTP error, so the whole curation
isn't lost to one 404.

## Privacy posture

- No analytics, no telemetry
- Keys in `localStorage` only (per-origin, per-user, per-browser)
- Outbound network traffic is limited to the configured image sources
- Only external dependency is JSZip 3.10.1, loaded from cdnjs

## Status

Early development. Expect breaking changes to the localStorage schema
between minor versions.

## License

All rights reserved. Code published here for development and review purposes.
