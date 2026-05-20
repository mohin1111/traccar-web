# useMapStyles.js

**Role:** Hook that returns the full array of 26 tile/style provider descriptors; each entry carries an `id`, human-readable `title`, MapLibre `style` URL or inline style object, `available` boolean, optional `transformRequest`, and optional `attribute` name indicating which server attribute key must be non-empty.
**Fits in:** Called only by `MapView.jsx` (line 60). The returned array drives both the `MapSwitcher` menu and the active-style selection logic.
**Read next:** [[MapView.jsx]] (consumes this), [[MapSwitcher.jsx]] (renders the picker menu).

## Public API
- Default export (line 32) — React hook. No arguments. Returns `StyleDescriptor[]` memoized on all API key preferences.

### StyleDescriptor shape
```js
{
  id: string,           // stable key used in localStorage / preferences
  title: string,        // translated label for MapSwitcher
  style: string | object, // MapLibre style URL or inline GL style object
  available: boolean,   // false = key not configured; hidden from switcher
  transformRequest?: (url) => { url },  // only ordnanceSurvey uses this
  attribute?: string,   // which server attribute enables this provider
}
```

## Key flows

### `styleCustom` helper (lines 6-30)
Builds a minimal MapLibre GL style object wrapping a single raster tile source. Used for every raster-only provider (OSM, Google, Bing, HERE, Yandex, etc.). Automatically deletes `undefined` keys so missing `minZoom`/`maxZoom`/`attribution` don't pollute the object.

### Free-key providers (always `available: true`)
OpenFreeMap, LocationIQ Streets/Dark (fallback key embedded), OSM, OpenTopoMap, Carto, Google Road/Satellite/Hybrid (unauthenticated tile URLs work without key), Yandex, AutoNavi, Ordnance Survey.

### Key-gated providers (`available: Boolean(key)`)
MapTiler Basic/Hybrid, Bing Road/Aerial/Hybrid, TomTom, HERE Basic/Hybrid/Satellite, Mapbox Streets/Dark/Outdoors/Satellite, Custom URL.

### Google Maps dual-mode (lines 99-143)
If `googleKey` is set, uses the `google://` protocol registered by `maplibre-google-maps` in `MapView.jsx`. If not set, falls back to direct tile CDN URLs — functional but potentially ToS-violating for production.

### Custom URL (lines 323-333)
If `customMapUrl` is a vector tile style JSON URL (no `{z}` / `{quadkey}` tokens), passes through verbatim. Otherwise wraps in `styleCustom()` as raster tiles. Populated from `state.session.server.mapUrl`.

### Ordnance Survey transform (lines 268-274)
The only provider using `transformRequest` — appends `&srs=3857` to every tile URL.

## Gotchas / non-obvious
- **`locationIqKey` has a hardcoded fallback** (line 38): `pk.0f147952a41c555a5b70614039fd148b`. This is Traccar's public demo key — rate-limited; replace in production.
- **`mapboxAccessToken` is read both here** (for Mapbox style URLs) **and in `MapView.jsx`** (line 69, set as `maplibregl.accessToken` globally). Must be consistent.
- **Memoization deps** (lines 335-345) include all API keys and `customMapUrl`. Adding a new provider must add its key to the dep array.
- **Google Hybrid** (line 132) uses the `satellite` tile type with `layerRoadmap` param — not `hybrid` type; tile server quirk.
- **AutoNavi** (line 253) Chinese provider; `minZoom: 3` set explicitly, tiles in Chinese (`lang=zh_cn`).

## Line index
- 6-30 — `styleCustom()` helper
- 32-45 — hook preamble: all key preferences
- 46 — start of the 26-item array
- 56 — `locationIqStreets` with fallback key
- 99-143 — Google Road/Satellite/Hybrid (dual-mode)
- 144-156 — MapTiler Basic/Hybrid
- 157-194 — Bing Road/Aerial/Hybrid
- 195-201 — TomTom
- 202-240 — HERE Basic/Hybrid/Satellite
- 241-251 — Yandex
- 252-265 — AutoNavi
- 266-274 — Ordnance Survey (only `transformRequest` user)
- 275-322 — Mapbox Streets/Dark/Outdoors/Satellite
- 323-334 — Custom URL
- 335-347 — `useMemo` dependency array
