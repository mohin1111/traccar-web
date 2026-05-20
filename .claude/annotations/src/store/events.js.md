# events.js

**Role:** Unread-event queue for the events drawer. Receives real-time event pushes from the WebSocket; capped at 50 items. UI can dismiss individual events or clear all.
**Fits in:** Written by `SocketController` (`eventsActions.add`). Read by `EventsDrawer` in `src/main/`. Alarm audio is played in `SocketController` before dispatching — this slice has no side effects.
**Read next:** [[SocketController.jsx]] (the writer), [[session.js]] (WS connection state that governs when events arrive)

## Public API
- `eventsActions.add(eventsArray)` — prepends events to front of queue (`unshift`); truncates to 50 (line 11)
- `eventsActions.delete({ id })` — removes a single event by `id`
- `eventsActions.deleteAll()` — clears the queue

## Key flows

### Cap at 50 (line 11)
`state.items.unshift(...action.payload)` adds new events at the front (newest first), then `state.items.splice(50)` drops everything beyond index 49. So only the 50 most recent events survive at any time.

### Event push from WS
`SocketController.onmessage` receives `data.events`, calls `handleEvents()` which:
1. Dispatches `eventsActions.add(events)` (if `features.disableEvents` is false)
2. Plays alarm audio if the event type/alarm matches user preferences
3. Sets `notifications` state for the Snackbar display

## Gotchas / non-obvious
- **Events are not persisted.** A page reload clears the queue. The events drawer is purely transient.
- **`features.disableEvents`** can suppress the add entirely — the events never enter the store. This is a server-configured feature flag.
- **`add` receives an array** (not a single event) — always pass `[event]` not `event`.

## Line index
- 5-7 — `initialState`: `items: []` (array, not map — events are ordered)
- 9-12 — `add`: unshift + splice(50)
- 13-15 — `delete`: filter by id
- 16-18 — `deleteAll`
