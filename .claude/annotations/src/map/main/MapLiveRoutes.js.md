# MapLiveRoutes.js

**Role:** Renders live breadcrumb trail polylines from `session.history` (accumulated positions from the WebSocket stream) for each visible device. Supports `none`/`selected`/`all` modes.
**Fits in:** Rendered by `MainMap.jsx`. Reads `session.history` (array of `[lng, lat]` coordinate pairs keyed by deviceId) and `devices.items`. Source/layer scoped by `useId()`.
**Read next:** [[MapView.jsx]] (map singleton), [[MapRouteCoordinates.js]] (similar polyline layer for reports/replay), [[MapPositions.js]] (markers at the current positions).

## Public API
- `MapLiveRoutes({ deviceIds })` (line 7, default export) — `deviceIds: number[]` — the subset of device IDs to render (from the filtered device list). Returns `null`.

## Key flows

### Mode guard (lines 22-58)
`type = useAttributePreference('mapLiveRoutes', 'none')`. If `type === 'none'`, no source/layer is created (returns early cleanup `() => {}`). When type is non-none, creates a GeoJSON source + line layer with data-driven `color`, `width`, `opacity` from feature properties.

### Data update (lines 61-95)
On `[type, devices, selectedDeviceId, history, deviceIds, ...]` change:
- Filters `deviceIds` to those that exist in `history` and `devices`.
- If `type === 'selected'`, further filters to only `selectedDeviceId`.
- `map.getSource(id)?.setData(FeatureCollection)` where each feature is a LineString of `history[deviceId]` coordinates.
- Color from `device.attributes['web.reportColor']` or `theme.palette.geometry.main`.

## Gotchas / non-obvious
- **`history[deviceId]`** is an array of `[lng, lat]` pairs maintained by `SocketController` — already in MapLibre coordinate order; no reversal needed.
- **Source creates a Feature (not FeatureCollection) initially** (line 27-33) then `setData` replaces it with a FeatureCollection. MapLibre accepts both.
- **Both effects depend on `type`** — when type changes to `none`, the cleanup of the first effect removes the source/layer; the second effect short-circuits on `if (type !== 'none')`. They're paired intentionally.
- **`mapLineWidth` and `mapLineOpacity`** are server attribute preferences — affect all live routes uniformly.

## Line index
- 12 — `type` preference (mapLiveRoutes: 'none'/'selected'/'all')
- 22-58 — source + layer creation effect (guarded by type !== 'none')
- 61-95 — data update effect
- 64-66 — device ID filtering (type + history + devices existence)
