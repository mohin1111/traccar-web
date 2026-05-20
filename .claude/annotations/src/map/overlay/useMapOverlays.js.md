# useMapOverlays.js

**Role:** Hook returning the array of 12 raster overlay descriptors (traffic, weather, rail, sea, custom); each entry carries an `id`, `title`, `source` (MapLibre raster source config), `available` boolean, and optional `attribute`.
**Fits in:** Called only by `MapOverlay.js`. Parallel to `useMapStyles.js` but for overlays drawn on top of the base map.
**Read next:** [[MapOverlay.js]] (sole consumer), [[useMapStyles.js]] (structural parallel for base tile styles).

## Public API
- Default export (line 20) — React hook. No arguments. Returns `OverlayDescriptor[]` memoized on API key preferences.

### OverlayDescriptor shape
```js
{
  id: string,
  title: string,
  source: { type: 'raster', tiles: string[], tileSize: 256, maxzoom?: number },
  available: boolean,
  attribute?: string,
}
```

## Key flows

### `sourceCustom` helper (lines 6-14)
Builds a raster source config object. Strips `undefined` keys. Used by all providers.

### `sourceOpenWeather` helper (lines 17-18)
Shorthand: builds a `sourceCustom` for OpenWeatherMap tile layers (`clouds_new`, `precipitation_new`, `pressure_new`, `wind_new`, `temp_new`). Returns `maxzoom: 18`.

### Providers (lines 29-132)
12 overlays: Google Traffic (requires `googleKey`), OpenSeaMap (free), OpenRailwayMap (free), OpenWeather Clouds/Precipitation/Pressure/Wind/Temperature (requires `openWeatherKey`), TomTom Flow/Incidents (requires `tomTomKey`), HERE Flow (requires `hereKey`), Custom URL (`state.session.server.overlayUrl`).

### Custom overlay (line 126-131)
`customMapOverlay` from `state.session.server.overlayUrl` (server config, not user preference). Unlike base map custom URL, this is always wrapped as raster tiles.

## Gotchas / non-obvious
- **Overlays are raster only** — no vector overlay support. The `MapOverlay.js` consumer adds a single `'raster'` type layer.
- **HERE Flow** (lines 113-124) uses four subdomain load-balanced URLs (`1.traffic.maps.ls.hereapi.com` through `4.traffic...`).
- **Google Traffic** requires the `google://` protocol registered by `maplibre-google-maps` in `MapView.jsx`; it won't work if `MapView.jsx` is not in the tree.
- **`customMapOverlay` from Redux** (`state.session.server.overlayUrl`) — different source from `customMapUrl` in `useMapStyles.js` which comes from `state.session.server.mapUrl`. Two separate server config keys.

## Line index
- 6-14 — `sourceCustom` helper
- 17-18 — `sourceOpenWeather` helper
- 20-28 — hook preamble: API key preferences + Redux overlayUrl
- 29 — start of 12-item array
- 30-39 — Google Traffic (googleKey required)
- 40-46 — OpenSeaMap (free)
- 47-52 — OpenRailwayMap (free)
- 53-87 — OpenWeather 5× layers
- 88-111 — TomTom Flow + Incidents
- 112-124 — HERE Flow
- 125-131 — Custom URL
