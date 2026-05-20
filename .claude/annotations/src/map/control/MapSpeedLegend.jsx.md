# MapSpeedLegend.jsx

**Role:** Adds a speed-color legend bar to the bottom-left of the map when positions with non-zero speed are present; shows a Turbo colormap gradient with min–max speed range text.
**Fits in:** Rendered by `MapRoutePoints.jsx` (conditionally, via `showSpeedControl` prop). Used on the replay page to explain the speed-color encoding of route point arrows.
**Read next:** [[MapRoutePoints.jsx]] (the only caller), [[MapRoutePath.js]] (uses the same color scale).

## Public API
- `MapSpeedLegend({ positions })` (line 22, default export) — `positions`: array of position objects with `.speed` (in knots). Returns `null`. Renders a MapLibre custom control.

## Key flows

### Gradient CSS (lines 10-13)
Module-level: generates 10 Turbo colormap stops via `interpolateTurbo(i/9)` from `colors.js`. Builds a CSS `linear-gradient` string. This is computed once at module load, not per render.

### Control lifecycle (lines 28-53)
`useEffect` on `[positions, speedUnit, t, theme.direction, classes.colorBar]`:
- Early return if `positions.length === 0` or `maxSpeed === 0`.
- On add: creates a div with `maplibregl-ctrl-scale` class, appends the gradient `colorBar` div and a `<span>` with `"min - max unit"` label text.
- Uses `speedFromKnots(speed, speedUnit)` + `speedUnitString(speedUnit, t)` for unit-aware display.
- Cleanup: `map.removeControl(control)`.

## Gotchas / non-obvious
- **No `useId()`** — the control is stateless DOM, not a MapLibre source/layer, so no ID needed.
- **`maxSpeed === 0` guard** (line 32): if all positions are stationary, the legend is hidden (zero-speed gradient is meaningless).
- **`classes.colorBar` in deps** (line 53): tss-react generates a stable class name per theme, so this rarely triggers re-creation.
- **Bottom-left placement** (line 51): mirrors the MapScale control position. RTL flips to bottom-right.

## Line index
- 10-13 — module-level Turbo gradient CSS generation
- 22 — component declaration
- 28-53 — control lifecycle effect
- 31-32 — early-return guards (empty / zero-speed)
- 43-44 — min/max speed with unit conversion
