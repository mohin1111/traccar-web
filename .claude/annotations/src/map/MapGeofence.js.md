# MapGeofence.js

**Role:** Read-only geofence display layer — renders all geofences from the Redux store as fill, line, and title symbol layers on the map. Respects the `mapGeofences` preference toggle and the `hide` attribute on individual geofences.
**Fits in:** Rendered by `MainMap.jsx` and `ReplayPage.jsx`. The display-only counterpart to `MapGeofenceEdit.js`. Source/layers scoped by `useId()`.
**Read next:** [[MapGeofenceEdit.js]] (editable version), [[mapUtil.js]] (`geofenceToFeature`, `findFonts`).

## Public API
- `MapGeofence()` (line 8, default export) — no props. Returns `null`. Reads everything from Redux/preferences.

## Key flows

### Layer setup (lines 17-78)
`useEffect` on `[mapGeofences, id]`:
- If `mapGeofences` is false: returns no-op cleanup (no source/layers created).
- Otherwise: creates GeoJSON source + 3 layers: `geofences-fill` (Polygon fill, 10% opacity, data-driven `color`), `geofences-line` (stroke, data-driven `color`/`width`/`opacity`), `geofences-title` (symbol, `{name}` text, `findFonts`).
- Layer IDs are **hardcoded** (`geofences-fill`, not `${id}-fill`) — only one `MapGeofence` per page, singleton usage.
- Cleanup checks and removes each layer + source.

### Data update (lines 80-89)
`useEffect` on `[mapGeofences, geofences, id, theme]`:
- Skips if `mapGeofences` false.
- Maps `Object.values(geofences)`, filters out `geofence.attributes.hide === true`, converts each via `geofenceToFeature(theme, geofence)`.
- Calls `map.getSource(id)?.setData(FeatureCollection)`.

## Gotchas / non-obvious
- **Source ID uses `useId()`** but **layer IDs are hardcoded** — this is an inconsistency in the codebase. The source ID doesn't matter externally; the layer IDs matter for `map.getLayer()` checks.
- **`geofenceToFeature`** sets `properties.color` from `geofence.attributes.color` or `theme.palette.geometry.main`. The fill and line layers use `['get', 'color']` expressions to read this.
- **`hide` attribute** (`geofence.attributes.hide`) — undocumented Traccar feature allowing individual geofences to be hidden on the map without deleting them.

## Line index
- 13 — `mapGeofences` preference (default true)
- 17-78 — source + 3 layers setup effect
- 26-35 — geofences-fill layer
- 37-45 — geofences-line layer
- 47-60 — geofences-title layer
- 62-75 — cleanup (layer + source removal)
- 80-89 — data update: filter hidden, convert, setData
