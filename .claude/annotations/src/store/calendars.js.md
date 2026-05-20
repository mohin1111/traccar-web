# calendars.js

**Role:** Client-side cache of calendar entities (id→calendar). Calendars define iCal recurrence schedules attached to notifications and maintenance rules.
**Fits in:** Written by `CachingController`. Read by notification and maintenance settings pages to present calendar pickers.
**Read next:** [[maintenances.js]] (uses calendars for recurrence scheduling), [[PACKAGE.md]] (full load sequence)

## Public API
- `calendarsActions.refresh(calendarsArray)` — replaces the entire map

## Key flows
`CachingController` fetches `GET /api/calendars` once on login. Calendar entities contain iCal-format `data` fields — the client stores them opaquely; parsing is done server-side or in the settings form.

## Gotchas / non-obvious
- **No `update` action.** Identical limitation to `groups.js`, `maintenances.js`.
- **Calendar data is base64-encoded iCal.** `items[id].data` is not human-readable JSON — it's an iCal string. Don't attempt to parse it client-side unless using an iCal library.

## Line index
- 5-7 — `initialState`: `items: {}`
- 9-12 — `refresh`
