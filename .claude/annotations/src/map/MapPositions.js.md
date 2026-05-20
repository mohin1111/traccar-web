# MapPositions.js

**Role:** The core live-tracking marker layer. Renders device positions as clustered symbol markers with direction arrows, separates the selected device onto its own source for z-ordering, handles marker/cluster/map click events, and drives the pointer-cursor CSS state.
**Fits in:** Rendered by `MainMap.jsx` and `ReplayPage.jsx`. The most complex component in `src/map/`. Reads `devices` and `selectedId` from Redux; receives `positions` as prop (live from WebSocket → Redux → `MainMap`).
**Read next:** [[MapView.jsx]] (map singleton + `mapImages` sprite keys), [[preloadImages.js]] (`mapIconKey` for category→icon name), [[MapSelectedDevice.js]] (camera follow), [[MapLiveRoutes.js]] (polylines from same positions).

## Public API
- `MapPositions({ positions, onMapClick, onMarkerClick, showStatus, selectedPosition, titleField, disabled })` (line 12, default export)
  - `positions: Position[]` — the full array to render.
  - `onMapClick(lat, lng)` — fired on background map click.
  - `onMarkerClick(positionId, deviceId)` — fired on marker click.
  - `showStatus: boolean` — if true, icon color encodes device status (`success`/`error`/`info`/`neutral`); if false, all `neutral`.
  - `selectedPosition` — position object used to determine which marker shows a direction arrow (in `'selected'` direction mode).
  - `titleField: string` — feature property key for text label (default `'name'`).
  - `disabled: boolean` — suppresses click callbacks (used during replay scrubbing).

## Key flows

### Source / layer setup (lines 108-221)
`useEffect` on layer-stable deps (`mapCluster`, `clusters`, callbacks, `iconScale`, `id`, `selected`, `titleField`):
1. Creates two GeoJSON sources: `id` (main, with `cluster: mapCluster, clusterMaxZoom: 14, clusterRadius: 50`) and `selected` (no cluster).
2. For each source: adds a symbol layer `{category}-{color}` icon pattern + text label (name/titleField), and a `direction-{source}` symbol layer with a direction arrow rotated to `['get', 'rotation']`.
3. Adds a cluster-count symbol layer on the main source (filter `['has', 'point_count']`).
4. Registers mouse enter/leave (cursor pointer), marker click, cluster click, and background map click event listeners.
5. Returns cleanup that removes all listeners, layers, and sources.

### Feature building (`createFeature`, lines 38-65)
`useCallback` on `[directionType, showStatus]`. Returns a GeoJSON feature properties object:
- `category`: `mapIconKey(device.category)` — normalizes to an icon name.
- `color`: if `showStatus` → `position.attributes.color || getStatusColor(device.status)`; else `'neutral'`.
- `direction`: true/false based on `directionType` preference (`'none'`/`'selected'`/`'all'`) and `position.course > 0`.
- `rotation`: `position.course` (degrees).

### Data update (lines 224-255)
`useEffect` on positions/devices/selectedPosition change:
- For each of `[id, selected]`: filters positions where `source === id` means `deviceId !== selectedDeviceId`, `source === selected` means `deviceId === selectedDeviceId`.
- Calls `map.getSource(source)?.setData(FeatureCollection)`.
- This is the hot path for live WebSocket updates.

### Cluster click (lines 91-106)
Async: `map.queryRenderedFeatures(event.point, { layers: [clusters] })` → gets `cluster_id` → `map.getSource(id).getClusterExpansionZoom(clusterId)` → `map.easeTo(center, zoom)`.

## Gotchas / non-obvious
- **Two-source split** (main + selected): ensures the selected device marker always renders on top (z-order). MapLibre renders features in source order; adding the selected device to a separate source added last keeps it above all others.
- **`disabledRef`** (line 35-36): click callbacks check `disabledRef.current` (not the `disabled` prop directly) — avoids re-registering click handlers on each `disabled` prop change.
- **`{category}-{color}` icon key** (line 133): resolved against `mapImages` sprite via MapLibre expression. If the key doesn't exist in sprites, the icon is invisible (no error). Always ensure `preloadImages` has run before this layer renders.
- **`onMapClick`** uses `!event.defaultPrevented` guard (line 72) — marker click handlers call `event.preventDefault()`, so background clicks only fire when no feature was clicked.
- **`'symbol-sort-key': ['get', 'id']`** (line 142) — sorts symbols by position ID within a tile to ensure deterministic rendering order for overlapping markers.

## Line index
- 12-20 — component declaration + props
- 21-23 — `id`, `clusters`, `selected` (three source/layer ID variants)
- 25-27 — `devices`, `selectedDeviceId` from Redux
- 32-36 — `disabledRef` stable-prop pattern
- 38-65 — `createFeature` factory
- 67-68 — cursor CSS callbacks
- 70-89 — map click + marker click callbacks
- 91-106 — cluster click (async getClusterExpansionZoom)
- 108-221 — layer/source setup + event binding effect
- 109-125 — two GeoJSON sources
- 126-165 — symbol + direction layers (looped over both sources)
- 167-179 — cluster count layer
- 181-184 — event registrations
- 186-215 — cleanup
- 224-255 — data update (hot path)
