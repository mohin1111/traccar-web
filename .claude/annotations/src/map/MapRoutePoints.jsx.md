# MapRoutePoints.jsx

**Role:** Renders clickable speed-colored triangle arrows (▲) at each position along a route; fires `onClick(positionId, index)` on click; optionally renders the `MapSpeedLegend` control.
**Fits in:** Used by `ReplayPage.jsx` alongside `MapRoutePath.js`. The `.jsx` extension distinguishes it as the one file in this route pair that may return JSX (the `MapSpeedLegend`).
**Read next:** [[MapRoutePath.js]] (companion path line layer), [[MapSpeedLegend.jsx]] (conditionally rendered speed legend), [[MapView.jsx]] (map singleton).

## Public API
- `MapRoutePoints({ positions, onClick, showSpeedControl })` (line 7, default export)
  - `positions: Position[]` — with `longitude`, `latitude`, `speed`, `course`, `id` fields.
  - `onClick(id, index)` — fired when a point arrow is clicked.
  - `showSpeedControl: boolean` — if true, renders `<MapSpeedLegend positions={positions} />`.
  Returns `null` or `<MapSpeedLegend>`.

## Key flows

### Layer setup (lines 24-64)
Adds a GeoJSON source + one symbol layer. Layer uses `text-field: '▲'` (triangle character) rotated by `['get', 'rotation']`, colored by `['get', 'color']` (text-color). `text-allow-overlap: true` shows all points even when dense. Registers click/hover listeners.

### Data update (lines 66-85)
Computes `minSpeed`/`maxSpeed`, maps positions to Point features with properties `{ index, id, rotation: course, color: getSpeedColor(speed, min, max) }`. Calls `setData`.

### Click handling (lines 12-22)
`onMarkerClick` reads `feature.properties.id` and `feature.properties.index` (the original array index) from the clicked feature. `event.preventDefault()` prevents the background `onMapClick` from also firing.

## Gotchas / non-obvious
- **`'▲'` character as icon** — uses a text symbol (Unicode arrow) rather than a sprite image for direction arrows. This is different from `MapPositions.js`'s `direction` sprite. Advantage: no image preloading; disadvantage: font-dependent rendering.
- **`text-color` driven by `['get', 'color']`** — the color is on the text paint property, not a standard `icon-color`. This works because the "icon" is actually a text character.
- **`showSpeedControl ? <MapSpeedLegend positions={positions} /> : null`** (line 87) — the only case in `src/map/` where a null-rendering component returns a child component as JSX.
- **`index` in feature properties** — the array position is stored so the replay scrubber can seek to the correct position when a point is clicked.

## Line index
- 7 — component declaration
- 10-11 — cursor CSS callbacks
- 12-22 — `onMarkerClick` callback
- 24-64 — source + symbol layer setup + event listeners + cleanup
- 66-85 — data update (speed colors)
- 87 — conditional MapSpeedLegend return
