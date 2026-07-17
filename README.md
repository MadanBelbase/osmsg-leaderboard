# OSMSG Leaderboard

A live, single-page leaderboard of OpenStreetMap contributors, powered by the [OSMSG](https://github.com/osgeonepal/osmsg) API (OSM Stats Generator, an OSGeo Nepal project). It shows who's mapping right now — filterable by hashtag and time range, auto-refreshing, installable as a PWA, with no build step or backend of its own.

## Features

### Leaderboard
- Sortable, searchable table of contributors with map changes, created/modified/deleted counts, changesets, and a visual created/modified/deleted mix bar per mapper
- Top-three podium with avatars pulled from openstreetmap.org
- Creators / Modifiers filter, pagination (10–100 rows), CSV export
- Per-user modal: rank, hashtags used, editor detection (iD, JOSM, Rapid, Vespucci, StreetComplete…), node/way/relation breakdown, and full tag-level contributions (buildings, highways, POIs, landuse, waterways, natural, amenities…)

### Overview
Totals strip for the selected window — created/modified/deleted, mappers, changesets — with an expandable detailed breakdown by element type and tag keys.

### Charts
- **Editors** — bar chart of the top 5 editors by users (from `/api/v1/editor-stats`)
- **Contributions by hashtag** — horizontal bars derived from the loaded contributor rows

Charts always reflect the current window: they clear on window change, empty results, or fetch failure, and a token guard drops out-of-order responses.

### Filtering & live updates
- Hashtag chips (multiple, OR-combined; empty = global stats)
- Time ranges: 1h, 24h, 7d, 30d, all-time, or a custom UTC range with a date-time picker
- Auto-refresh every 60 s with a connection status pill (click to pause/resume/retry)
- Filters and range are written to the URL, so views are shareable/bookmarkable

## File structure

```
osmsg-leaderboard/
├── index.html            layout: form, window bar, overview, podium, table, modal
├── app.js                config + utils + state + API layer + transforms + UI (single bundle)
├── charts.js             editors and hashtag charts + hideCharts()
├── style.css             theme (cream/green)
├── sw.js                 Workbox service worker (offline cache, PWA)
└── manifest.webmanifest  PWA manifest
```

`index.html` loads `charts.js` then `app.js` as plain deferred scripts — no ES modules, no bundler, works from any static server.

### Endpoints used

| Endpoint | Used for | Params sent |
|---|---|---|
| `GET /api/v1/stats` | Leaderboard, podium, overview, hashtag chart | `start`, `end`, repeated `hashtag` |
| `GET /api/v1/editor-stats` | Editors chart | `start`, `end`, `limit=100` |
| `GET /health` | "Server updated X ago" chip | — |

Also available in the API layer (not yet rendered): `fetchHashtagStats` (`/api/v1/hashtag-stats`) and `fetchMapCentroids` (`/api/v1/map`).

### External services

| Service | Purpose |
|---|---|
| `api.openstreetmap.org` | User avatars and per-user editor detection (cached) |
| `cdn.jsdelivr.net` | Chart.js, Lucide icons, flatpickr |
| `cdn.tailwindcss.com` | Tailwind (CDN build; fine for this static page, swap for a compiled build in production if desired) |
| `fonts.googleapis.com` | Fraunces, Plus Jakarta Sans, JetBrains Mono |

## Running locally

ES modules aren't used, but the service worker requires http(s), so serve the folder:

```bash
npx serve .
# or
python3 -m http.server 8000
```

Open http://localhost:8000.

**After updating files:** the service worker caches HTML/JS (stale-while-revalidate). Hard-reload with `Ctrl+Shift+R`, or unregister the worker (Firefox: `about:debugging#/runtime/this-firefox`; Chrome: DevTools → Application → Service Workers) to make sure the new code is served.

