# `src/store/` — Redux store (the Traccar data contract)

Redux Toolkit store for the Traccar web frontend. Each slice maps 1:1 to a Traccar REST entity returned by `/api/<entity>`. **This directory is the single most important read for Transport OS backend design** — every field this store holds is a field the server must produce.

## Data contract framing

The slices below define what the Traccar server guarantees. When building the Transport OS backend on top of Traccar server, these shapes are the API contract our service layer must satisfy (or extend):

| Slice | Server entity | REST endpoint | WS push? |
|---|---|---|---|
| `session.js` | `server` + `user` + live `positions` | `GET /api/server`, `GET /api/session`, `GET /api/positions` | `{positions}` + `{logs}` |
| `devices.js` | `Device` | `GET /api/devices` | `{devices}` |
| `events.js` | `Event` | — | `{events}` |
| `geofences.js` | `Geofence` | `GET /api/geofences` | — |
| `groups.js` | `Group` | `GET /api/groups` | — |
| `drivers.js` | `Driver` | `GET /api/drivers` | — |
| `maintenances.js` | `Maintenance` | `GET /api/maintenances` | — |
| `calendars.js` | `Calendar` | `GET /api/calendars` | — |
| `motion.js` | (client-computed) | — | — |
| `errors.js` | (client-only) | — | — |

## File index

| File | One-liner | Annotation |
|---|---|---|
| `index.js` | Store config: `combineReducers` + `configureStore` + throttle middleware | [[index.js]] |
| `session.js` | Richest slice: `server`, `user`, socket status, live `positions` map, `history`, `logs` | [[session.js]] |
| `devices.js` | `items` (id→device) + `selectedId`; refreshed on WS + on initial load | [[devices.js]] |
| `events.js` | Unread event queue (capped at 50); receives WS `{events}` push | [[events.js]] |
| `geofences.js` | id→geofence lookup; cached by `CachingController` | [[geofences.js]] |
| `groups.js` | id→group lookup; cached by `CachingController` | [[groups.js]] |
| `drivers.js` | uniqueId→driver lookup; keyed by `uniqueId` (not `id`) | [[drivers.js]] |
| `maintenances.js` | id→maintenance lookup; cached by `CachingController` | [[maintenances.js]] |
| `calendars.js` | id→calendar lookup; cached by `CachingController` | [[calendars.js]] |
| `motion.js` | Transient motion state set by `MotionController`; clears on unmount | [[motion.js]] |
| `errors.js` | Global FIFO error queue; consumed by UI error banner | [[errors.js]] |
| `throttleMiddleware.js` | Adaptive rate-limiter for `update(devices)` + `updatePositions`; prevents UI freeze at high update rates | [[throttleMiddleware.js]] |

## Dependency graph

```
index.js
 ├── all 10 reducers
 └── throttleMiddleware
      └── imports sessionActions + devicesActions (to filter by type)

SocketController (src/)
 └── dispatches → devices.update, session.updatePositions,
                  events.add, session.updateLogs, devices.refresh

CachingController (src/)
 └── dispatches → geofences.refresh, groups.refresh,
                  drivers.refresh, maintenances.refresh, calendars.refresh

App (src/)
 └── dispatches → session.updateUser

ServerProvider (src/common/)
 └── dispatches → session.updateServer
```

## Session bootstrap sequence

1. `ServerProvider` renders → `GET /api/server` → `sessionActions.updateServer(serverObj)`
2. `App` renders (protected route) → `GET /api/session` → `sessionActions.updateUser(userObj)` or redirect `/login`
3. `CachingController` → parallel `GET` for geofences/groups/drivers/maintenances/calendars → `*.refresh(items[])`
4. `SocketController` → `GET /api/devices` → `devicesActions.refresh(devices[])` → opens WebSocket
5. WS messages arrive → throttleMiddleware → devices/positions/events/logs slices

## Multi-file flows

### Live position update (high frequency)
1. WS message `{positions: [...]}` → `SocketController.onmessage`
2. `dispatch(sessionActions.updatePositions(positions))`
3. `throttleMiddleware` intercepts: if > 3 dispatches/s, buffers + dedupes by `deviceId`; flushes on adaptive timer
4. `session.positions[deviceId]` updated; `session.history[deviceId]` appended (ring buffer, configurable length)
5. `MapPositions.js` reads `session.positions` and calls `map.getSource(id).setData(...)` — no React re-render

### Device status change
1. WS `{devices: [...]}` → `devicesActions.update(devices[])` (also throttled)
2. `devices.items[id]` updated in place
3. `DeviceList` re-renders via `useSelector`

### Reconnect / REST fallback
On WS close (non-logout): `SocketController` polls `GET /api/devices` + `GET /api/positions`, dispatches `devicesActions.update` + `sessionActions.updatePositions`, then waits 60 s before reconnecting.

## Conventions

- **id→item maps, not arrays.** All reference data slices use `{}` not `[]`, keyed by `id` (except `drivers` which uses `uniqueId`).
- **`refresh` vs `update`**: `refresh` replaces the entire map (full reload); `update` upserts by id.
- **No selectors file.** All `useSelector` calls inline `(state) => state.<slice>.<field>` at usage sites.
- **All slice actions exported from `index.js`** — consumers `import { fooActions } from './store'`, not from the slice file directly (except `eventsActions` which some files import from `./store/events` directly).

## How to add a new entity slice

1. Create `src/store/<entity>.js` with `createSlice`, export `<entity>Actions` + `<entity>Reducer`.
2. Add reducer to `combineReducers` in `index.js`; re-export actions.
3. Fetch in `CachingController` or `SocketController` and dispatch `<entity>Actions.refresh(items)`.
4. If the entity receives WS push, handle the key in `SocketController.onmessage`.
