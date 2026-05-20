# devices.js

**Role:** Holds the full device registry (`items`: id→device) and tracks which device is currently selected (`selectedId`). Receives both full refreshes (on login) and incremental updates (via WebSocket).
**Fits in:** Written by `SocketController` (initial `refresh` + incremental `update`). Read by `DeviceList`, `MainPage`, `MapPositions`, and most settings pages.
**Read next:** [[session.js]] (holds the live positions that pair with devices here), [[throttleMiddleware.js]] (intercepts `update`)

## Public API
- `devicesActions.refresh(devicesArray)` — wipes `items` and repopulates from array; used on initial authenticated load
- `devicesActions.update(devicesArray)` — upserts each device by `id`; used for WS incremental push
- `devicesActions.selectId(id)` — sets `selectedId` + stamps `selectTime`; used for map focus
- `devicesActions.remove(id)` — deletes a single device from `items`

## Key flows

### Initial load vs incremental update
`SocketController` calls `devicesActions.refresh` once on authentication (after `GET /api/devices`). Subsequent WS `{devices}` messages dispatch `devicesActions.update`, which only upserts — it never clears the map. This means a device deleted server-side won't disappear from the client until next `refresh` (full reload or re-login).

### `selectId` stamps `selectTime` (line 18)
`state.selectTime = Date.now()` is set alongside `selectedId`. This timestamp lets map camera logic distinguish a user-initiated selection from a pre-existing selection, enabling one-time fly-to behavior without re-triggering on every re-render.

## Gotchas / non-obvious
- **`update` action is intercepted by `throttleMiddleware`** — under high WS load, multiple `update` dispatches are coalesced; only the latest object per `id` survives the merge window.
- **`remove` is not throttled** — it bypasses `throttleMiddleware` and fires immediately.
- **`selectedId` is `null` not `undefined` initially** — prefer `=== null` checks, not falsy checks, since device id `0` is theoretically valid.

## Line index
- 5-8 — `initialState`: `items: {}`, `selectedId: null`
- 10-13 — `refresh` (full wipe + repopulate)
- 14-16 — `update` (upsert by id)
- 17-20 — `selectId` (sets id + selectTime)
- 21-23 — `remove` (delete by id)
