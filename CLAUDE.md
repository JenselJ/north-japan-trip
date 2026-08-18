# north-japan-trip — conventions

Winter Japan trip, **30 Jan – 13 Feb 2026**, 7 travellers.

`locations.geojson` is the single source of truth for places. `index.html` renders it as a
Leaflet map. **Never hardcode a place into the HTML** — the map reads the GeoJSON at runtime,
so adding a place means editing only `locations.geojson`.

## What must never be committed

**This repo is public and world-readable, deliberately so.** Anyone can read every file and
every past commit, so treat all content here as published.

Never commit:

- **Booking or reservation confirmation numbers** — hotel, ryokan, flight PNRs, rail passes,
  tour bookings, car hire.
- **Personal phone numbers** — anyone's, traveller or contact.

Fine to commit: hotel and venue names, street addresses, opening hours, prices, and the
dates we plan to be somewhere. Those are all public information already.

If real reservation details genuinely need to live in the repo, they go in
**`bookings.private.md`**, which is gitignored and must stay that way. Never `git add -f` it,
and never move its contents into a tracked file.

Note that git history is permanent: a confirmation number committed and then deleted in a
later commit is still readable in the repo's history. If something sensitive does get
committed, say so rather than quietly deleting it — it needs the history rewritten and the
booking treated as compromised.

## File roles

| File | Role |
|---|---|
| `locations.geojson` | The data. Every place lives here. |
| `index.html` | Leaflet map, Leaflet via CDN, no build step. Reads `locations.geojson`. |
| `README.md` | Human-facing overview + live map URL. |

## Property schema

Every feature is a GeoJSON `Point`. `geometry.coordinates` is **`[longitude, latitude]`** —
that order, not lat/lon. All 12 properties below are required; use `""` for unknown text
fields rather than omitting the key.

| Property | Type | Notes |
|---|---|---|
| `id` | string | Slug, `city-name`, e.g. `niigata-ponshukan`. Unique across the file. |
| `name` | string | English/romaji name. |
| `name_ja` | string | Japanese name. `""` if genuinely unknown. |
| `category` | string | Exactly one of the six below. |
| `city` | string | English city name, e.g. `Morioka`. |
| `cost` | string | Free text, e.g. `¥500`, `TBC`. |
| `access` | string | How to get there from the nearest station. |
| `booking` | string | e.g. `Not required — walk in`, `Book 2 weeks ahead`. |
| `stars` | number | `0` or `1`. `1` = must-do; renders a ★ in the popup. |
| `dates` | string | Which trip day(s) this is pencilled in for. `""` if unscheduled. |
| `notes` | string | Anything else worth knowing. |
| `added` | string | ISO date `YYYY-MM-DD` the entry was added. |

## Categories — fixed list

A category must be **exactly** one of these strings (they key the marker colours in
`index.html`; a typo silently drops the place into "Uncategorised"):

- `Food & drink`
- `Anime`
- `Onsen & sentō`
- `Unique / rare`
- `Festivals & events`
- `Round1`

Adding a seventh category means also adding its colour to `CATEGORY_COLOURS` in `index.html`.
Note `sentō` and `Unique / rare` carry non-ASCII and spaced-slash spellings — copy them verbatim.

## Rules for adding a place

1. **Real coordinates only — never guess, never approximate from memory.**
   Geocode the actual address (Nominatim, OSM, an official site) and sanity-check that the
   result falls in the right city before saving. Reverse-check the returned address.
   Chains and multi-branch venues are the trap: "Ponshukan" resolves to a branch in
   Yuzawa as readily as the Niigata Station one. Bound the search to the intended city.
2. **Unique slug ids.** Check the id does not already exist before adding:
   `python3 -c "import json;d=json.load(open('locations.geojson'));print([f['properties']['id'] for f in d['features']])"`
   Never add a feature whose `id` is already present.
3. **Keep features sorted by `city`, then `category`**, so diffs stay readable.
4. **Validate before committing:** the file must be valid JSON, ids unique, every `category`
   in the fixed list, and every coordinate inside Japan's bounding box
   (lon 122–154, lat 24–46).
5. **Commit and push after any edit**, one place per commit where practical:
   ```
   git commit -m "add <name> (<city>)"
   git push
   ```

## Previewing locally

`index.html` fetches `locations.geojson`, so opening it as a `file://` URL fails on CORS.
Serve it instead:

```
python3 -m http.server 8899   # then open http://localhost:8899/
```

The page already shows an on-map error box explaining this if the fetch fails.
