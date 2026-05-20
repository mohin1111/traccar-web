# throttleMiddleware.js

**Role:** Redux middleware that rate-limits `devicesActions.update` and `sessionActions.updatePositions` dispatches. Prevents UI thread from freezing when many devices report positions simultaneously. Uses an adaptive flush interval tied to actual render time.
**Fits in:** Appended to the middleware chain in `index.js`. Intercepts only the two high-frequency action types; all others pass through immediately.
**Read next:** [[index.js]] (where it's attached), [[session.js]] (`updatePositions` is the primary target), [[devices.js]] (`update` is the secondary target)

## Public API
The middleware is a factory function (`export default () => (next) => (action) => ...`). No external configuration — constants are file-scoped:
- `threshold = 3` — dispatches per second before throttling kicks in
- `minInterval = 1500` ms — minimum flush interval
- `maxInterval = 30000` ms — maximum flush interval
- `scaleFactor = 1000` — multiplier applied to flush render time to compute next interval

## Key flows

### Pass-through for non-targeted actions (lines 65-70)
Any action whose type is not `devicesActions.update` or `sessionActions.updatePositions` bypasses the buffer entirely and goes straight to `next(action)`.

### Throttle detection + counter (lines 72-85)
Each targeted action increments `counter`. On the `tick` timer cycle: if `(counter * 1000) / currentInterval > threshold` → throttling activates. In throttle mode, the action is buffered instead of forwarded.

### `tick` — adaptive flush loop (lines 19-61)
Runs on `setTimeout` at `currentInterval`. Each cycle:
1. If throttled: flush the entire buffer. Coalesces by id — only the latest device/position per id survives (lines 25-34).
2. Dispatches one merged `update` action for devices and one for positions.
3. Measures flush render time (`performance.now()` delta).
4. Recalculates `currentInterval = clamp(renderTime * scaleFactor, minInterval, maxInterval)` — slow renders → longer interval → fewer flushes per second.
5. Re-evaluates `throttled` flag for the next cycle.

### Deduplication during buffer flush (lines 25-34)
`deviceUpdates[item.id] = item` and `positionUpdates[item.deviceId] = item` — last-writer-wins per id. If the same device reported 20 positions during the buffer window, only the most recent position is dispatched.

## Gotchas / non-obvious
- **Events in the buffer window are permanently lost** — there is no backfill. Only the latest position per device is forwarded. This is intentional (map only needs current state).
- **`throttled` starts as `false`** — the first few dispatches always pass through immediately. Throttling only activates after the threshold is crossed.
- **`currentInterval` self-tunes to render cost.** On a fast machine with few devices, `minInterval = 1500` ms. On a slow machine with 1 000 devices and a slow flush, it may reach 30 s.
- **`devMode` logging** (`debugLog`) is active in development — expect console output during local development.
- **Two separate `setTimeout` chains** are not used — a single `tick` recursively reschedules itself with `setTimeout(tick, currentInterval)` (line 60).
- **`performance.now()`** measures time from navigation start, not epoch — the delta between two calls gives elapsed ms, which is what matters here.

## Line index
- 4-7 — throttle constants
- 10-11 — debug logging helpers
- 13 — middleware factory signature
- 14-18 — per-invocation state: buffer, throttled flag, counter, currentInterval
- 19-61 — `tick` function (the adaptive flush loop)
- 23 — buffer splice (take all + clear)
- 25-34 — dedup by id (last-writer-wins)
- 46-50 — adaptive interval recalculation
- 53-58 — throttle flag toggle
- 60 — recursive setTimeout reschedule
- 65-70 — pass-through for non-targeted action types
- 72 — counter increment
- 74-78 — buffer push when throttled
- 80-85 — inline throttle-start detection (before tick fires)
