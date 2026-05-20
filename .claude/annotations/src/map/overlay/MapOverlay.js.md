# MapOverlay.js

**Role:** Reads the active overlay preference, finds the matching overlay descriptor from `useMapOverlays`, and adds/removes a single raster source+layer on the global map accordingly.
**Fits in:** Rendered by `MainMap.jsx`. Thin layer between `useMapOverlays` (data) and the MapLibre source/layer API.
**Read next:** [[useMapOverlays.js]] (provides overlay descriptors), [[MapView.jsx]] (map singleton).

## Public API
- `MapOverlay()` (line 6, default export) — no props. Returns `null`.

## Key flows

### Active overlay selection (lines 10-14)
`selectedMapOverlay = useAttributePreference('selectedMapOverlay')`. Filters `mapOverlays` to `available` ones (drops those with missing API keys), then finds the one matching `selectedMapOverlay`. If no match or no value set, `activeOverlay` is `undefined`.

### Source + layer lifecycle (lines 16-36)
`useEffect` on `[id, activeOverlay]`:
- If `activeOverlay` is defined: `map.addSource(id, activeOverlay.source)` + `map.addLayer({ id, type: 'raster', source: id, layout: { visibility: 'visible' } })`.
- Cleanup always attempts `map.removeLayer(id)` + `map.removeSource(id)` (guarded).
- When `activeOverlay` is `undefined` (no overlay selected), the effect body does nothing and the cleanup is a no-op — effectively hiding the overlay.

## Gotchas / non-obvious
- **Single overlay at a time** — only one `MapOverlay` instance renders, so layer ID from `useId()` is unique per mount. No conflict risk.
- **Overlay layer sits on top of all base map layers** but below MapLibre control UI. Layer insertion order: MapLibre adds new layers on top by default.
- **`activeOverlay.source` is a raw raster source config** (the full object from `useMapOverlays`) — passed directly to `map.addSource`. No wrapping needed.

## Line index
- 6 — component declaration
- 10 — `selectedMapOverlay` preference
- 12-14 — find active overlay from available list
- 16-36 — source + layer effect (add on truthy, always cleanup)
