# MapAccuracy.js

**Role:** Renders semi-transparent accuracy circles (GPS uncertainty halos) around each position by converting accuracy radius in meters to Turf circle polygons and pushing them to a fill layer.
**Fits in:** Rendered by `MainMap.jsx` and `ReplayPage.jsx`, receiving the current position list. Source/layer are scoped by `useId()` to avoid conflicts.
**Read next:** [[MapView.jsx]] (map singleton), [[MapPositions.js]] (renders the markers at the same positions).

## Public API
- `MapAccuracy({ positions })` (line 6, default export) — `positions`: array of `{ longitude, latitude, accuracy }` objects. `accuracy` in meters. Returns `null`.

## Key flows

### Layer setup (lines 11-38)
On mount: adds a GeoJSON source (empty FeatureCollection) and a `fill` layer with `filter: ['all', ['==', '$type', 'Polygon']]`. Fill color and outline both `theme.palette.geometry.main` at 25% opacity.

### Data update (lines 41-50)
On `positions` change: maps positions where `accuracy > 0` to `turfCircle([lon, lat], accuracy * 0.001)` (converts meters to km for Turf). Calls `map.getSource(id)?.setData(FeatureCollection)`.

### Cleanup (lines 31-37)
Checks `map.getLayer(id)` and `map.getSource(id)` before removing — defensive pattern used throughout `src/map/`.

## Gotchas / non-obvious
- **`accuracy * 0.001`** — Turf `circle` default unit is km; accuracy is in meters. This is the unit conversion.
- **`useId()`** generates a React-stable unique string (e.g. `:r1:`). MapLibre source/layer IDs accept arbitrary strings, so this works correctly.
- **`theme.palette.geometry.main`** is a custom palette key added by Traccar's theme file — not a standard MUI color. Must be present when vendoring.

## Line index
- 11-38 — source + fill layer setup + cleanup
- 41-50 — positions → Turf circles → setData
