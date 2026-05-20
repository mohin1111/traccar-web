# MapRuler.jsx

**Role:** Interactive distance-measurement tool; adds a ruler button control, three MapLibre layers (`ruler-line`, `ruler-point`, `ruler-label`), and click-to-add-point behavior with optional snapping to existing positions. Shows cumulative distance as a label on the last point.
**Fits in:** Rendered by `ReplayPage.jsx` (which passes `positions` for snap targets). Manages its own source/layers on the global `map` singleton.
**Read next:** [[MapView.jsx]] (map singleton), [[mapUtil.js]] (`findFonts`), [[MapSpeedLegend.jsx]] (similar control pattern).

## Public API
- `MapRuler({ positions, onActiveChange })` (line 28, default export)
  - `positions` — array of `{ longitude, latitude }` objects used for snap-to-existing-point (15 px snap radius).
  - `onActiveChange(active: bool)` — called when ruler is toggled on/off. Parent uses this to disable other map interactions.

## Key flows

### Layer setup (lines 46-78)
On mount: adds `ruler` GeoJSON source (empty FeatureCollection), then three layers: line (LineString filter), point (Point filter), label (symbol with `has: label` filter). Layer color taken from `theme.palette.geometry.main`.

### Snap logic (lines 80-88)
`snap(lngLat, pixel)` iterates `positionsRef.current`, projects each position with `map.project()`, checks Euclidean pixel distance < 15. If snap found returns `[lon, lat]` of nearest position; otherwise returns raw click coords.

### Click accumulation + render (lines 90-117)
Each map click (when active) pushes a snapped coordinate to `points[]` then calls `render()`. `render()` computes cumulative distance via `maplibregl.LngLat.distanceTo()`, builds a FeatureCollection of Points (last one gets a `label` property with formatted total distance) and optionally a LineString, then calls `map.getSource('ruler').setData(...)`.

### Toggle (lines 119-130)
Button click flips `active` bool, toggles `'active'` CSS class, calls `onActiveChange`. When deactivated: clears `points`, calls `render()` (empties the source), removes click listener.

### Cleanup (lines 154-168)
Removes control, deregisters click listener if still active, removes all three layers then the source. Guards each with `map.getLayer(id)` checks.

## Gotchas / non-obvious
- **`positionsRef` and `onActiveChangeRef`** are ref-forwarded props — the single effect captures everything in the outer closure; refs allow props to update without re-running the effect (and re-adding source/layers).
- **`distanceUnit` is not in the effect dep array** (line 170) — the effect intentionally recreates if `theme`, `t`, `distanceUnit`, or `classes.button` changes. This means the ruler resets when unit preference changes.
- **`formatDistance(total, distanceUnit, t)`** — the `t` here is for unit suffix translation, not distance value formatting.
- **Source name is hardcoded `'ruler'`** (not `useId()`) — only one ruler can exist at a time; no conflict risk since the component is a singleton on any given page.

## Line index
- 28 — component declaration
- 40-43 — refs for stable props
- 46-78 — source + 3 layers creation
- 80-88 — `snap()` pixel-distance check
- 90-116 — `render()` distance calculation + setData
- 114-117 — click handler
- 119-130 — `toggle()` button handler
- 132-152 — control injection (createRoot + onAdd/onRemove)
- 154-168 — cleanup
