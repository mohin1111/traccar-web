# errors.js

**Role:** Global FIFO error queue. Errors are pushed from any component or async handler; the UI error banner pops and displays them one at a time.
**Fits in:** Written from anywhere via `errorsActions.push(message)`. Read by the top-level error banner component (likely in `src/common/` or `App`).
**Read next:** [[index.js]] (store assembly), [[session.js]] (session errors are typically shown here)

## Public API
- `errorsActions.push(messageString)` — appends an error message to the queue
- `errorsActions.pop()` — removes the oldest error (FIFO); no-op if queue is empty

## Key flows

### FIFO queue pattern
`push` appends to the back (line 9: `push`). `pop` removes from the front (line 11: `shift`). The UI banner typically calls `pop()` after displaying, creating a sequential display of accumulated errors.

## Gotchas / non-obvious
- **Queue is unbounded.** If errors accumulate faster than the UI dismisses them, the queue grows indefinitely. In practice this is rare since errors are user-visible.
- **Payload is expected to be a string.** No structured error type — just the message.

## Line index
- 5-7 — `initialState`: `errors: []`
- 9-11 — `push` (appends to back)
- 12-15 — `pop` (shifts from front, guarded against empty)
