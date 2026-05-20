# MapScale.js

**Role:** Adds a MapLibre built-in scale bar (`ScaleControl`) to the map; updates its unit system (`metric`/`imperial`/`nautical`) based on the `distanceUnit` user preference.
**Fits in:** Rendered by `MainMap.jsx`. Trivial wrapper; the only behavior is the unit switching logic.
**Read next:** [[MapView.jsx]] (map singleton), [[MapSpeedLegend.jsx]] (also placed bottom-left).

## Public API
- `MapScale()` (line 7, default export) — no props. Returns `null`.

## Key flows

### Control add (lines 14-17)
`useMemo` creates one `ScaleControl` instance. `useEffect` on `[control, theme.direction]` adds it to `bottom-left` (LTR) or `bottom-right` (RTL). Cleanup removes it.

### Unit switching (lines 19-32)
`useEffect` on `[control, distanceUnit]`: maps `'mi'` → `imperial`, `'nmi'` → `nautical`, default → `metric`. Calls `control.setUnit(...)`.

## Gotchas / non-obvious
- **`ScaleControl` is memoized** — same instance is reused across unit changes so the control doesn't remount (which would flash the UI). `setUnit` mutates it in place.
- **Bottom-left placement** shared with `MapSpeedLegend` — in practice `MapSpeedLegend` appears on replay pages and `MapScale` on all pages; they don't conflict because MapLibre stacks bottom-left controls vertically.

## Line index
- 10 — `distanceUnit` preference
- 12 — `ScaleControl` useMemo
- 14-17 — addControl / removeControl effect
- 19-32 — unit switching effect
