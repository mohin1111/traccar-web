# maintenances.js

**Role:** Client-side cache of maintenance schedule entities (id→maintenance). Loaded at login; used by `MotionController` and maintenance report/settings pages.
**Fits in:** Written by `CachingController`. Read by `MotionController` to compute maintenance due status and by `MaintenancesPage`/`MaintenancePage` for CRUD.
**Read next:** [[motion.js]] (consumes maintenances to compute motion state), [[calendars.js]] (same shape)

## Public API
- `maintenancesActions.refresh(maintenancesArray)` — replaces the entire map

## Key flows
`CachingController` fetches `GET /api/maintenances` once on login. Maintenance entities carry type + period + start fields that `MotionController` evaluates against odometer/hours to flag overdue status.

## Gotchas / non-obvious
- **No `update` action.** Settings page saves require a full refresh or navigation to trigger re-fetch.

## Line index
- 5-7 — `initialState`: `items: {}`
- 9-12 — `refresh`
