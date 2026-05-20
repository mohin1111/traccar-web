# MapRoutePath.js

**Role:** Renders a speed-color-coded route path for replay/report pages by splitting positions into per-segment LineString features, each colored by `getSpeedColor` (Turbo scale). Falls back to a solid `web.reportColor` if the device has one configured.
**Fits in:** Used by `ReplayPage.jsx` and `PositionsReportPage.jsx`. A companion to `MapRoutePoints.jsx` (which renders the clickable point arrows on the same route).
**Read next:** [[MapRoutePoints.jsx]] (route arrows + speed legend, shares the same positions), [[MapRouteCoordinates.js]] (solid-color variant for reports), [[mapUtil.js]] (no direct use, but shares the coordinate convention).

## Public API
- `MapRoutePath({ positions })` (line 8, default export) — `positions: Position[]` with `longitude`, `latitude`, `speed`, `deviceId` fields. Returns `null`.

## Key flows

### Layer setup (lines 30-64)
Adds a GeoJSON source + one line layer (`${id}-line`) with data-driven `color`/`width`/`opacity`. Line join/cap are `round`.

### Data update (lines 66-91)
On positions change:
1. Computes `minSpeed` and `maxSpeed` across all positions.
2. Builds `features[]`: for each consecutive pair `[i, i+1]`, one LineString feature with `color: reportColor || getSpeedColor(positions[i+1].speed, min, max)`.
3. Calls `setData(FeatureCollection(features))`.

Speed-color is applied to the **end point** of each segment (`positions[i+1]`), not the start. This gives a gradient effect as speed changes.

### Color fallback
`reportColor` is `null` if the device has no `web.reportColor` attribute → `getSpeedColor(...)` from `colors.js` (Turbo scale). If `reportColor` is set → solid color for the whole route (no speed gradient).

## Gotchas / non-obvious
- **Segments, not a single LineString** — the per-segment approach is required for per-segment color encoding. A single MultiLineString would require paint expressions that MapLibre doesn't support for per-feature color on a single LineString source.
- **`reportColor` check is first** (line 81): if the device attribute is set, `getSpeedColor` is never called — the Turbo scale is bypassed entirely.
- **`MapRoutePath` + `MapRoutePoints`** are typically rendered together; the path shows the colored line, the points show clickable arrows at each position.

## Line index
- 8 — component declaration
- 13-25 — `reportColor` resolution from Redux device attributes
- 30-64 — source + line layer setup + cleanup
- 66-91 — data update (segment feature generation)
- 67-68 — min/max speed computation
- 70-86 — per-segment feature loop
