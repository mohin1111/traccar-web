# MapCamera.js

**Role:** Generic imperative camera helper; fits the map to a set of positions or coordinates (via `fitBounds`), or jumps to a single lat/lon (via `jumpTo`). Runs on every prop change — no guard against re-fires.
**Fits in:** Used by report and replay pages that need to frame a result set on load (e.g. `PositionsReportPage`, `ReplayPage`). Not used on the main live-tracking page (which uses `MapDefaultCamera` instead).
**Read next:** [[MapDefaultCamera.js]] (initialized-once variant), [[MapSelectedDevice.js]] (device-follow camera).

## Public API
- `MapCamera({ latitude, longitude, positions, coordinates })` (line 5, default export)
  - `coordinates?: [lng, lat][]` — pre-formatted coordinate array (takes priority over `positions`).
  - `positions?: { longitude, latitude }[]` — raw position objects (converted internally).
  - `latitude, longitude` — fallback single-point jump target.
  Returns `null`.

## Key flows

### Branch logic (lines 7-26)
Single `useEffect` on all four props:
- **`coordinates` or `positions` provided**: reduces to `LngLatBounds`, calls `map.fitBounds(bounds, { padding: 10%, duration: 0 })`.
- **Empty array**: early return (no-op, bounds would be invalid).
- **Neither**: `map.jumpTo({ center: [longitude, latitude], zoom: max(current, 10) })`.

## Gotchas / non-obvious
- **`duration: 0`** on `fitBounds` — instant jump, no animation. Different from `MapSelectedDevice`'s `easeTo`.
- **`coordinates[0]` used as both LngLatBounds constructor args** (line 12): `new LngLatBounds(coords[0], coords[0])` creates a zero-extent bounds, then `reduce` extends it. This is the correct pattern for a reduce-based bounds computation.
- **No `initialized` guard** — every prop change re-fires the camera. Callers must only pass this component while the data is stable.
- **`positions` takes `[item.longitude, item.latitude]` order** (line 8) — correct for MapLibre `[lng, lat]`.

## Line index
- 5 — component declaration + props
- 7-26 — single effect: fitBounds or jumpTo branch
- 8 — positions → [lng, lat] map
- 10-19 — fitBounds path
- 21-25 — jumpTo path
