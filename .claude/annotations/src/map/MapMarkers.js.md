# MapMarkers.js

**Role:** Generic static marker layer — renders an array of `{ latitude, longitude, image, title }` markers as MapLibre symbol features, optionally with title text labels.
**Fits in:** Used by `MapRoutePoints.jsx` for start/end route markers, and by any page needing simple icon overlays. Not for clustered live devices (use `MapPositions.js`). Source/layer scoped by `useId()`.
**Read next:** [[MapPositions.js]] (the clustered, status-aware variant), [[preloadImages.js]] (`mapImages` keys used for `image` prop values).

## Public API
- `MapMarkers({ markers, showTitles })` (line 8, default export)
  - `markers: { latitude, longitude, image?: string, title?: string }[]` — `image` defaults to `'default-neutral'` if not provided.
  - `showTitles?: boolean` — if true, adds a `text-field: '{title}'` layout to the symbol layer.
  Returns `null`.

## Key flows

### Layer setup (lines 15-67)
`useEffect` on `[showTitles, iconScale, id]`: adds GeoJSON source + one symbol layer. If `showTitles`: layout includes `text-field: '{title}'`, `text-anchor: 'bottom'`, `text-offset: [0, -2*iconScale]`. If not: only icon, no text.

### Data update (lines 69-84)
`useEffect` on `[showTitles, markers, id]`: maps `markers` to GeoJSON Point features with `image` and `title` as properties. Calls `map.getSource(id)?.setData(FeatureCollection)`.

### Icon scale (lines 13)
`iconScale = useAttributePreference('iconScale', desktop ? 0.75 : 1)` — responsive default: smaller on desktop, full-size on mobile.

## Gotchas / non-obvious
- **No clustering** — unlike `MapPositions.js`, no `cluster: true` on the source. All markers always visible.
- **`showTitles` in the data update deps** (line 84) — appears redundant (titles are a layout property, not a data property) but is included to force a data refresh when the layer structure changes, preventing stale text state.
- **`filter: ['!has', 'point_count']`** (line 29) — included even though there is no cluster, as a defensive no-op (leaves the door open for clustering to be added later).

## Line index
- 8 — component declaration
- 13 — `iconScale` preference
- 15-67 — source + layer setup (showTitles branch)
- 24-44 — showTitles=true layer definition
- 46-56 — showTitles=false layer definition
- 59-66 — cleanup
- 69-84 — markers → features → setData
