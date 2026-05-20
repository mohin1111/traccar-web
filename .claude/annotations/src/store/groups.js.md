# groups.js

**Role:** Client-side cache of group entities (id→group). Loaded once at login; used by device list filtering and settings pages.
**Fits in:** Written by `CachingController` (`GET /api/groups`). Read by `DeviceList` for group-filter UI and `GroupPage` for editing.
**Read next:** [[geofences.js]] (identical shape), [[devices.js]] (devices belong to groups via `groupId` field)

## Public API
- `groupsActions.refresh(groupsArray)` — replaces the entire map; no incremental `update` action

## Key flows
`CachingController` fetches once on login and dispatches `refresh`. No WS push; no `update` action — settings pages that modify groups call `refresh` after save or simply reload the list.

## Gotchas / non-obvious
- **Only `refresh`, no `update`.** Unlike `geofences.js`, there is no upsert action. A settings page that adds/edits a group must trigger a full `refresh` or navigate away and back.
- **Group hierarchy is flat in this slice.** Parent-child group relationships (via `groupId` on Group entity) are resolved at usage sites, not in the store.

## Line index
- 5-7 — `initialState`: `items: {}`
- 9-13 — `refresh` (wipe + repopulate)
