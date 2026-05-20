# session.js

**Role:** The richest slice. Holds global auth context (`server`, `user`), WebSocket connection status, live positions map (deviceId→latest position), live-route history, and optional server-streamed logs.
**Fits in:** Written by `ServerProvider` (server), `App` (user), `SocketController` (positions/logs/socket). Read by virtually every component that needs auth context or live tracking data.
**Read next:** [[throttleMiddleware.js]] (intercepts `updatePositions`), [[devices.js]] (parallel live-data slice), [[SocketController.jsx]] (the writer of positions/logs)

## Public API
- `sessionActions.updateServer(serverObj)` — sets `state.server`
- `sessionActions.updateUser(userObj)` — sets `state.user`; `null` signals logged-out
- `sessionActions.updateSocket(bool)` — sets `state.socket` (true=connected)
- `sessionActions.enableLogs(bool)` — toggles `state.includeLogs`; clears `state.logs` when disabled
- `sessionActions.updateLogs(logsArray)` — appends to `state.logs`
- `sessionActions.updatePositions(positionsArray)` — upserts positions + maintains history ring buffer

## Key flows

### `updatePositions` — live position upsert with history (lines 33-55)
This is the most complex reducer in the entire store:
1. Reads `liveRoutes` preference from `state.user.attributes` falling back to `state.server.attributes` (default `'none'`).
2. Reads `liveRoutesLimit` (default 10) the same way.
3. For each incoming position: `state.positions[deviceId] = position` (always).
4. If `liveRoutes !== 'none'`: appends `[lng, lat]` to history ring buffer, deduplicated by coordinate equality; ring buffer is sliced to `liveRoutesLimit` length.
5. If `liveRoutes === 'none'`: resets `state.history = {}` (clears all history).

### `enableLogs` — log streaming toggle (lines 24-29)
Clears accumulated `state.logs` when logs are disabled. This prevents stale log entries from being shown if logs are re-enabled later.

## Gotchas / non-obvious
- **`positions` is a map, not an array.** Key is `deviceId` (number). Map lookup `O(1)` from map components is intentional.
- **`history` dedup uses coordinate equality only** (line 45): `last[0] !== position.longitude && last[1] !== position.latitude` — if a device is stationary, the history ring doesn't grow.
- **`liveRoutes` / `liveRoutesLimit` read inside the reducer.** This is unusual (reducers normally don't read cross-slice state) but valid in RTK because both keys are in the same slice's state (`state.user` and `state.server` are both in the session slice).
- **`history` ring buffer uses `[lng, lat]` not `[lat, lng]`** (line 46) — MapLibre / GeoJSON coordinate order (longitude first). Don't reverse these.
- **`socket: null` initial state** (not `false`) means "never connected" vs `false` = "lost connection".
- **`updatePositions` is intercepted by `throttleMiddleware`** — the reducer may not fire immediately on every WS message; multiple calls are coalesced by deviceId.

## Line index
- 5-13 — `initialState` shape (the server-entity fields documented inline)
- 15-17 — `updateServer`
- 18-20 — `updateUser`
- 21-23 — `updateSocket`
- 24-29 — `enableLogs` (clears log array on disable)
- 30-32 — `updateLogs` (appends)
- 33-55 — `updatePositions` (the complex one: upsert + history ring buffer)
- 35-39 — liveRoutes + limit read from user/server attributes
- 43-49 — history dedup + ring buffer append
- 50-53 — history clear when liveRoutes is 'none'
