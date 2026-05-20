# drivers.js

**Role:** Client-side cache of driver entities. Uniquely keyed by `uniqueId` (not numeric `id`) to enable fast lookup by the identifier sent from the vehicle device.
**Fits in:** Written by `CachingController` (`GET /api/drivers`). Read by position detail views and driver report pages to resolve a device's assigned driver.
**Read next:** [[groups.js]] (same load pattern), [[PACKAGE.md]] (data contract overview)

## Public API
- `driversActions.refresh(driversArray)` — replaces the entire map; keyed by `item.uniqueId`

## Key flows
`CachingController` fetches once on login. The `uniqueId` key (line 11: `state.items[item.uniqueId] = item`) matches the driver identifier transmitted in device position attributes (`driverUniqueId`), enabling `O(1)` lookup of driver name/details from a position.

## Gotchas / non-obvious
- **Keyed by `uniqueId`, not `id`.** This is the only slice where the map key is not the numeric entity `id`. Accessing `state.drivers.items[numericId]` will always return `undefined` — use the string `uniqueId`.
- **No `update` action.** Same limitation as `groups.js`.

## Line index
- 5-7 — `initialState`: `items: {}`
- 9-12 — `refresh` (wipe + repopulate, keyed by `uniqueId`)
