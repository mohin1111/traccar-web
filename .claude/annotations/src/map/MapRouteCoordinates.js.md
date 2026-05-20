# MapRouteCoordinates.js

**Role:** Renders a named polyline route from an explicit coordinate array onto the map, with a title label symbol. Used by report pages to display a single device's route as a solid-color line (not speed-colored).
**Fits in:** Used by `CombinedReportPage.jsx` and similar multi-device report views where each device route has a distinct color. Distinct from `MapRoutePath.js` (speed-colored, single device) and `MapLiveRoutes.js` (live, no labels).
**Read next:** [[MapRoutePath.js]] (speed-colored variant), [[MapLiveRoutes.js]] (live trail variant), [[MapView.jsx]] (map singleton).

## Public API
- `MapRouteCoordinates({ name, coordinates, deviceId })` (line 8, default export)
  - `name: string` — displayed as the route label via the title symbol layer.
  - `coordinates: [lng, lat][]` — pre-formatted coordinate pairs.
  - `deviceId: number` — used to look up `web.reportColor` device attribute. Falls back to `theme.palette.geometry.main`.
  Returns `null`.

## Key flows

### Layer setup (lines 27-77)
Adds a GeoJSON source (empty LineString), then two layers:
- `${id}-line`: data-driven `color`/`width`/`opacity` from feature properties.
- `${id}-title`: symbol layer with `text-field: '{name}'`, 12px, `findFonts(map)`.

### Data update (lines 80-94)
Sets the source to a single Feature (LineString) with `properties: { name, color: reportColor, width: mapLineWidth, opacity: mapLineOpacity }`.

### Color resolution (lines 13-23)
`reportColor = useSelector(...)` — checks `state.devices.items[deviceId].attributes['web.reportColor']`. If set: uses it; else `theme.palette.geometry.main`.

## Gotchas / non-obvious
- **Two layers per instance** (`${id}-line` and `${id}-title`) vs. `MapRoutePath.js`'s one layer — the title label is unique to this component.
- **Coordinates must already be `[lng, lat]`** — no reversal. Callers must convert from Traccar's `{ longitude, latitude }` format.
- **`mapLineWidth` and `mapLineOpacity` are server attribute preferences** — apply uniformly across all route visualizations.

## Line index
- 8 — component declaration
- 13-23 — `reportColor` from device attribute or theme fallback
- 27-65 — source + two layers setup
- 67-77 — cleanup
- 80-94 — LineString data update
