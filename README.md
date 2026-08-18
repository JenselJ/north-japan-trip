# north-japan-trip

Planning repo for a winter trip to northern Japan — **30 January to 13 February 2027**, seven
of us. It exists so that everyone's suggestions land in one place instead of scattering across
chat threads.

`locations.geojson` is the source of truth: every place we're considering is a point feature
with its cost, access, booking situation, notes, and which days it's pencilled in for.
`index.html` renders that file as a Leaflet map — markers coloured by category (food & drink,
anime, onsen & sentō, unique / rare, festivals & events, and Round1), each category toggleable,
with a ★ on the must-dos. Adding a place means editing the GeoJSON only; the map picks it up
automatically.

**Live map:** _not yet published — see below._

To preview locally, serve the folder rather than opening the file directly (the map fetches the
GeoJSON, which browsers block over `file://`):

```
python3 -m http.server 8899
```

then open <http://localhost:8899/>.

Conventions for adding places — the property schema, the fixed category list, and the
rule that coordinates must always be looked up rather than guessed — are in `CLAUDE.md`.
