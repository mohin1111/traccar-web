# geofences.js

**Role:** Client-side cache of all geofence entities (id→geofence). Loaded once at login by `CachingController`; used by the map geofence layer and geofence report pages.
**Fits in:** Written by `CachingController` (`GET /api/geofences`). Read by `MapGeofence.js` (renders fill/line/label layers on the map) and `GeofencesPage`.
**Read next:** [[groups.js]] (same shape), [[PACKAGE.md]] (how CachingController populates all reference caches)

## Public API
- `geofencesActions.refresh(geofencesArray)` — replaces the entire map with a fresh fetch result
- `geofencesActions.update(geofencesArray)` — upserts by id (used after CRUD operations in settings)

## Key flows

### Full-replace on login
`CachingController` fetches `GET /api/geofences` once after authentication and dispatches `refresh`. There is no incremental WS push for geofences — settings-page saves call `update` directly after a successful PUT/POST response.

## Gotchas / non-obvious
- **No WS push.** Geofence changes made by another user in a parallel session are not reflected until reload.
- **`update` does not remove.** Deleted geofences linger in `items` until next `refresh`.

## Line index
- 5-7 — `initialState`: `items: {}`
- 9-12 — `refresh` (full replace)
- 13-15 — `update` (upsert by id)
