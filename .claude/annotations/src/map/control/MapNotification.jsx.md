# MapNotification.jsx

**Role:** Renders a bell-icon control button in the MapLibre control bar; toggles an `active` CSS class (red bell) based on the `enabled` prop; calls the `onClick` prop when pressed.
**Fits in:** Rendered by `MainMap.jsx` to drive the events drawer toggle. Purely a styled button bridge between React state and the MapLibre control bar.
**Read next:** [[MapView.jsx]] (provides `map` singleton), [[MapGeocoder.jsx]] (same control-injection pattern), [[MapSwitcher.jsx]] (same pattern).

## Public API
- `MapNotification({ enabled, onClick })` (line 22, default export) — `enabled: bool` toggles the red-highlight state; `onClick: () => void` fires when the button is pressed. Returns `null` (no JSX rendered in the React tree).

## Key flows

### Stable `onClick` via ref (lines 26-28)
`onClickRef` is updated each render but the event handler always calls `onClickRef.current()` — avoids stale-closure issues without re-creating the MapLibre control on each prop change.

### Active state toggle (lines 57-59)
`useEffect` on `enabled` calls `buttonRef.current?.classList.toggle('active', enabled)`. The `&&.active` CSS rule in `useStyles` colors the bell icon `theme.palette.error.main` when `active` is present.

### Control injection (lines 31-55)
Standard MapLibre custom control pattern. Button hosts `NotificationsIcon` via `createRoot`. Cleanup uses `queueMicrotask(() => root.unmount())`.

## Gotchas / non-obvious
- **`buttonRef`** is populated inside `onAdd` — guaranteed to be set before the `enabled` effect runs (effects run after mount, `onAdd` runs when `map.addControl` is called which is inside the same effect). Order is safe.
- Returns `null` — this component has no React DOM output; its entire UI is injected via MapLibre's control mechanism.

## Line index
- 22 — component declaration + props
- 26-28 — `onClickRef` stable-closure pattern
- 31-55 — control injection effect
- 57-59 — `enabled` → classList toggle
