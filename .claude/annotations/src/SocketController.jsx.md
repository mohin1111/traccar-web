# SocketController.jsx

**Role:** Manages the WebSocket lifecycle for real-time data. On authentication: fetches devices, opens `ws[s]://host/api/socket`, dispatches incoming messages into Redux, plays alarm audio, shows event Snackbars. On disconnect: runs REST poll fallback and schedules a 60 s reconnect. Also reconnects on `window.online` and tab visibility restoration.
**Fits in:** Rendered (renderless) inside `<App>` immediately after auth. The sole writer of `session.updatePositions`, `devices.update/refresh`, and `events.add`. Closes the socket with code 4000 on logout.
**Read next:** [[session.js]] (`updatePositions` target), [[devices.js]] (`update/refresh` target), [[events.js]] (`add` target), [[throttleMiddleware.js]] (intercepts the high-frequency dispatches from here)

## Public API
`SocketController` is a default-exported renderless React component. Returns Snackbar notifications for events that carry `attributes.message`. No props.

## Key flows

### Initial authenticated setup (`useAsyncTask`, lines 147-162)
Runs when `authenticated` flips to true:
1. `GET /api/devices` → `devicesActions.refresh(devices)` (full replace, not update)
2. `nativePostMessage('authenticated')` — notifies Flutter manager wrapper
3. `connectSocket()` — opens the WebSocket
4. Cleanup: `clearReconnectTimeout()` + `socket.close(logoutCode)` (code 4000 signals intentional logout; `onclose` handler skips reconnect on this code)

### WebSocket message dispatch (`onmessage`, lines 124-138)
Each message is a JSON object; fields are processed independently:
- `data.devices` → `devicesActions.update(devices)` (throttled)
- `data.positions` → `sessionActions.updatePositions(positions)` (throttled)
- `data.events` → `handleEventsRef.current(events)` (immediate)
- `data.logs` → `sessionActions.updateLogs(logs)` (immediate)

### Event handling (`handleEvents`, lines 53-76)
1. `eventsActions.add(events)` — unless `features.disableEvents`
2. Alarm audio: plays if any event type is in `soundEvents` pref, or if type is `'alarm'` and alarm subtype is in `soundAlarms` pref (default: `'sos'`)
3. Sets `notifications` local state → renders Snackbar for each event with a `message` attribute

### Disconnect + REST fallback (`onclose`, lines 96-122)
1. `dispatch(sessionActions.updateSocket(false))`
2. If code !== 4000: poll `GET /api/devices` + `GET /api/positions`; dispatch updates
3. If either returns 401 → `navigate('/login')`
4. Schedule `connectSocketRef.current()` after 60 s

### Reconnect on network/visibility (lines 187-212)
`window.online` event and `visibilitychange` (tab becoming visible) both call `reconnectIfNeeded()`:
- If socket is CLOSED → `connectSocket()`
- If socket is OPEN → sends `'{}'` (keep-alive ping to test connection)

### Log streaming toggle (`useEffect`, lines 143-145)
When `session.includeLogs` changes, sends `JSON.stringify({ logs: includeLogs })` to the open socket. Server toggles log streaming in response.

### `connectSocketRef` indirection (lines 81, 141)
`connectSocket` is created with `useCallback`. `connectSocketRef.current = connectSocket` is assigned after creation. `onclose` calls `connectSocketRef.current?.()` rather than `connectSocket` directly — avoids stale-closure capture inside the `setTimeout`.

## Gotchas / non-obvious
- **`handleEventsRef`** (lines 78-79) same pattern as `connectSocketRef` — ensures the `onmessage` closure always calls the latest version of `handleEvents` without re-creating the socket.
- **Alarm audio is a module-level singleton** (`alarmAudio`, line 20) — created once on first play and reused. `currentTime = 0` before `play()` allows replaying before the previous playback finishes.
- **Close code 4000** (`logoutCode`, line 18) is the agreed signal for intentional logout. The `onclose` handler short-circuits on this code and skips REST fallback + reconnect.
- **Race guard in `onclose`** (lines 101, 106, 116): `if (socketRef.current !== socket) return` — prevents a stale `onclose` from a superseded socket from writing stale data or scheduling a new reconnect after `connectSocket()` was already called externally.
- **Native notification bridge** (lines 164-179): `handleNativeNotificationListeners` is a Set; `SocketController` registers `handleNativeNotification` which fetches the event by id from `/api/events/:id` and feeds it into `handleEvents`. Used by the Flutter manager app for FCM-delivered event notifications.
- **`devicesActions.refresh` on initial load, `update` on WS push.** `refresh` wipes the map; `update` upserts. Don't swap them — `update` on first load would leave stale devices from a prior session.

## Line index
- 18 — `logoutCode = 4000`
- 20-27 — alarm audio singleton + `playAlarm`
- 33-34 — `authenticated` + `includeLogs` selectors
- 48-49 — `soundEvents` / `soundAlarms` attribute preferences
- 53-76 — `handleEvents` (add to store + alarm + Snackbar)
- 83-139 — `connectSocket` callback (the main WS management function)
- 88-89 — WebSocket URL construction
- 92-94 — `onopen` → `updateSocket(true)`
- 96-122 — `onclose` → REST fallback + reconnect timer
- 124-138 — `onmessage` → dispatch to slices
- 141 — `connectSocketRef.current = connectSocket` (stale-closure fix)
- 143-145 — log toggle send
- 147-162 — `useAsyncTask` initial authenticated setup
- 164-179 — native notification bridge handler
- 182-185 — native listener registration
- 187-212 — online/visibility reconnect listeners
- 214-226 — JSX: Snackbar renders for event notifications
