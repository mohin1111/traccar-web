# motion.js

**Role:** Transient client-computed state slice. Holds a map of device motion/maintenance status as computed by `MotionController` from live positions and maintenance schedules. Not a server entity.
**Fits in:** Written exclusively by `MotionController` (`src/main/MotionController.jsx`). Read by `DeviceList` / `DeviceRow` to show maintenance-due indicators.
**Read next:** [[maintenances.js]] (input to MotionController), [[session.js]] (positions — the other input), [[devices.js]] (devices iterated by MotionController)

## Public API
- `motionActions.set(itemsObject)` — replaces the entire map wholesale (not per-device upsert)
- `motionActions.clear()` — resets to `{}`; called on `MotionController` unmount

## Key flows

### Full replace on each position update
`MotionController` re-runs computation across all devices whenever positions change and dispatches `set(newMap)` with the complete result. There is no per-device incremental update — the whole map is swapped each time.

### `clear` on unmount
`MotionController` calls `motionActions.clear()` in its cleanup, preventing stale motion state from appearing if the controller re-mounts.

## Gotchas / non-obvious
- **Not a server entity.** Nothing in this slice maps to a Traccar REST response — it is a pure client-side derivation.
- **`set` replaces entirely** — not an upsert. If `MotionController` only evaluates a subset of devices, devices absent from the new map lose their motion state.
- **The shape of `items` values** is defined by `MotionController`, not here — the slice is a generic `{}` container.

## Line index
- 5-7 — `initialState`: `items: {}`
- 9-11 — `set` (full replace)
- 12-14 — `clear` (reset to empty)
