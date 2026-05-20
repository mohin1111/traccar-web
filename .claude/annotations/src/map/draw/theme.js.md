# theme.js

**Role:** Static data file — an exact copy of the `@mapbox/mapbox-gl-draw` v1.4.0 default theme array, exported as the default. Used by `MapGeofenceEdit.js` as the base draw style layer list.
**Fits in:** Imported only by `MapGeofenceEdit.js` (line 13). No React, no dependencies.
**Read next:** [[MapGeofenceEdit.js]] (sole consumer; appends a `gl-draw-title` symbol layer on top of this).

## Public API
- Default export (line 4) — `LayerDefinition[]`. 17 layer definitions covering polygon fill/stroke (inactive, active, static), line (inactive, active, static), midpoint circles, vertex circles, and point circles. Two color states: inactive=`#3bb2d0` (teal), active=`#fbb03b` (orange).

## Key flows
No logic — pure static export. The only modification from the original source is: it lives in this repo rather than being imported from the `mapbox-gl-draw` package, allowing future customization without forking the package.

## Gotchas / non-obvious
- **Hardcoded hex colors** (`#3bb2d0`, `#fbb03b`, `#fff`, `#404040`) — not theme-aware. If the app theme changes, draw colors do not follow.
- **The comment on line 1-2** references v1.4.0 of the upstream source. If `@mapbox/mapbox-gl-draw` is upgraded, this file may drift from the new default and produce visual inconsistencies.
- **`userProperties: true`** must be set on the `MapboxDraw` instance (which it is in `MapGeofenceEdit.js`) for the `{user_name}` expression in the custom title layer to work. This file itself does not enforce that.

## Line index
- 4 — start of 17-item layer array
- 5-19 — `gl-draw-polygon-fill-inactive`
- 20-29 — `gl-draw-polygon-fill-active`
- 30-36 — `gl-draw-polygon-midpoint`
- 37-56 — polygon stroke inactive/active
- 57-88 — line inactive/active
- 103-120 — vertex circles
- 170-215 — static mode styles
