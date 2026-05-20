# MapGeofenceEdit.js

**Role:** Full CRUD geofence editor; integrates `@mapbox/mapbox-gl-draw` (patched for MapLibre CSS classes) onto the global map, syncs draw state with Redux and the Traccar REST API, and navigates to the settings page on create.
**Fits in:** Rendered by `GeofencesPage.jsx`. Depends on `react-router-dom` (`useNavigate`), `react-redux` (geofences slice), `reactHelper` (`useCatchCallback`), and `fetchOrThrow`. The only file in `src/map/` that writes to the server.
**Read next:** [[mapUtil.js]] (`geofenceToFeature`, `geometryToArea`), [[theme.js]] (draw styles array), [[MapGeofence.js]] (read-only display counterpart).

## Public API
- `MapGeofenceEdit({ selectedGeofenceId })` (line 21, default export) — `selectedGeofenceId?: number` — when set, flies the map to that geofence's bounding box. Returns `null`.

## Key flows

### MapboxDraw CSS patch (lines 17-19)
Overwrites MapboxDraw internal CSS class constants to use `maplibregl-*` prefixes instead of `mapboxgl-*`. Must run before the `new MapboxDraw()` call; happens at module evaluation time.

### Draw instance (lines 27-56)
`useMemo([], [])` creates the `MapboxDraw` once. Config: polygon + line_string + trash controls; `userProperties: true` (exposes custom props in feature.properties). Styles = base `drawTheme` + a custom `gl-draw-title` symbol layer that renders `{user_name}` (the geofence name) on each feature using `findFonts(map)`.

### Geofence population (lines 132-137)
`useEffect` on `[geofences, draw, theme]`: calls `draw.deleteAll()` then re-adds every geofence via `draw.add(geofenceToFeature(theme, geofence))`. This is a full refresh on any geofence store change.

### Create flow (lines 72-92)
`map.on('draw.create', listener)`:
1. Converts drawn feature geometry to WKT via `geometryToArea(feature.geometry)`.
2. Deletes the drawn feature from draw state immediately (`draw.delete(feature.id)`).
3. POSTs to `/api/geofences` with `{ name: t('sharedGeofence'), area: wkt }`.
4. On success: navigates to `/settings/geofence/${item.id}`.

### Update flow (lines 109-130)
`map.on('draw.update', listener)`: finds the matching geofence in Redux state by `feature.id === i.id`, PUTs updated area, then `refreshGeofences()`.

### Delete flow (lines 94-107)
`map.on('draw.delete', listener)`: DELETEs by `feature.id`, then `refreshGeofences()`.

### Selected geofence camera (lines 139-153)
When `selectedGeofenceId` changes: gets feature from draw, reduces vertices to `LngLatBounds`, calls `map.fitBounds` with 10% canvas padding.

## Gotchas / non-obvious
- **`draw.add()` uses `feature.id` as the geofence `id`** (numeric). MapboxDraw normally assigns string IDs, but `geofenceToFeature` sets `id: item.id` — this makes `draw.get(selectedGeofenceId)` work with the numeric Traccar id.
- **The CSS patch at lines 17-19 is fragile**: tied to MapboxDraw v1.x internal constant names. Upgrading MapboxDraw may break it silently (controls appear but unstyled).
- **`refreshGeofences`** is a `useCatchCallback` wrapper around a `fetchOrThrow('/api/geofences')` + `dispatch(geofencesActions.refresh(...))` — it re-fetches the full list, not a delta.
- **`findFonts(map)` is called inside `useMemo`** (line 45): safe because `map` singleton has a style loaded by the time `GeofencesPage` renders (it's behind `mapReady`).

## Line index
- 17-19 — MapboxDraw CSS class constants patch
- 27-56 — `useMemo` draw instance creation
- 60-63 — `refreshGeofences` async callback
- 65-70 — mount: `refreshGeofences()` + `map.addControl(draw, ...)`
- 72-92 — `draw.create` handler (POST + navigate)
- 94-107 — `draw.delete` handler (DELETE)
- 109-130 — `draw.update` handler (PUT)
- 132-137 — geofence population effect
- 139-153 — `selectedGeofenceId` camera effect
