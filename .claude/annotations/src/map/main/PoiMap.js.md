# PoiMap.js

**Role:** Fetches and renders a KML point-of-interest layer from a user-configured URL by converting KML to GeoJSON (`@tmcw/togeojson`) and adding fill, point, line, and title symbol layers to the map.
**Fits in:** Rendered by `MainMap.jsx`. Active only when the `poiLayer` preference (server attribute) is set. Uses `useAsyncTask` for fetch abort. Source/layers use hardcoded IDs (`poi-fill`, `poi-point`, `poi-line`, `poi-title`).
**Read next:** [[MapView.jsx]] (map singleton), [[mapUtil.js]] (`findFonts`), [[MapGeofence.js]] (similar multi-layer pattern).

## Public API
- `PoiMap()` (line 9, default export) — no props. Returns `null`.

## Key flows

### KML fetch (lines 18-27)
`useAsyncTask({ signal })` — fetches `poiLayer` URL with abort signal (cancels on `poiLayer` change or unmount), parses with `DOMParser` as XML, converts with `kml(dom)` to GeoJSON FeatureCollection, stores in `data` state.

### Layer creation (lines 29-99)
`useEffect` on `[data, id, theme.palette.geometry.main]`:
- Adds a single GeoJSON source with the full FeatureCollection.
- Adds 4 layers: `poi-fill` (Polygon fill using KML `fill`/`fill-opacity` properties), `poi-point` (circle for non-polygon points using `icon-color`), `poi-line` (LineString using `stroke`/`stroke-width`/`stroke-opacity`), `poi-title` (symbol with `{name}` text field).
- Returns cleanup that removes all layers + source.

### KML property passthrough
KML extended style attributes are preserved as GeoJSON feature properties by `@tmcw/togeojson`. The `['coalesce', ['get', 'prop'], fallback]` expressions use them if present, otherwise fall back to `theme.palette.geometry.main`.

## Gotchas / non-obvious
- **Hardcoded layer IDs** (`poi-fill` etc.) — not `useId()`-scoped. Only one `PoiMap` can exist per page, which is acceptable since it's always a singleton.
- **`useAsyncTask`** from `reactHelper` wraps async work with automatic abort on cleanup — the `signal` is passed to `fetch`.
- **`poiLayer` is a plain URL string** from server preferences — no authentication. CORS must allow the Traccar web origin to fetch the KML.

## Line index
- 14 — `poiLayer` preference
- 18-27 — KML fetch with abort
- 29-99 — layer creation effect
- 35-42 — poi-fill layer
- 44-52 — poi-point (circle) layer
- 53-62 — poi-line layer
- 63-79 — poi-title symbol layer
- 80-98 — cleanup (layer + source removal)
