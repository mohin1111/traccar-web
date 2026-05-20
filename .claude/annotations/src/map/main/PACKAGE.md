# `src/map/main/` — Live-tracking map behaviors

Five behavior components that run on the main live-tracking page (`MainMap.jsx`). All return `null`; all read from Redux state. None manage map controls — they only manage sources, layers, and camera.

## File index

| File | One-liner | Annotation |
|---|---|---|
| `MapAccuracy.js` | GPS accuracy halos via Turf circle polygons (fill layer) | [MapAccuracy.js.md](MapAccuracy.js.md) |
| `MapDefaultCamera.js` | One-time camera initialization (selected device → defaults → auto-fit) | [MapDefaultCamera.js.md](MapDefaultCamera.js.md) |
| `MapLiveRoutes.js` | Breadcrumb polylines from `session.history` (none/selected/all) | [MapLiveRoutes.js.md](MapLiveRoutes.js.md) |
| `MapSelectedDevice.js` | Camera follow on device selection / position update (easeTo) | [MapSelectedDevice.js.md](MapSelectedDevice.js.md) |
| `PoiMap.js` | KML POI layer fetched from `poiLayer` preference URL | [PoiMap.js.md](PoiMap.js.md) |

## Redux dependencies

| Component | Redux reads |
|---|---|
| `MapDefaultCamera.js` | `devices.selectedId`, `session.positions` |
| `MapLiveRoutes.js` | `devices.items`, `devices.selectedId`, `session.history` |
| `MapSelectedDevice.js` | `devices.selectedId`, `devices.selectTime`, `session.positions` |
| `MapAccuracy.js` | none (receives `positions` as prop) |
| `PoiMap.js` | none (reads `poiLayer` from preferences hook → `session.server.attributes`) |

## Camera precedence

`MapDefaultCamera` (one-shot, `initialized` gate) → `MapSelectedDevice` (ongoing, `easeTo` on selection/follow).
`MapCamera.js` (in `src/map/`) is the third camera variant used by report/replay pages instead of these two.

## Conventions

- All five use `useId()` for source IDs where applicable (`MapAccuracy`, `MapLiveRoutes`, `PoiMap`).
- `MapLiveRoutes` and `MapAccuracy` guard against `type === 'none'` / empty positions before creating sources.
- `PoiMap` is the only file in this directory with a network fetch — via `useAsyncTask` (abort on unmount/change).
- `MapSelectedDevice` is the only file in this directory that reads `usePrevious` values for change detection.
