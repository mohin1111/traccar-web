# MapDefaultCamera.js

**Role:** Handles the one-time initial camera position on app load — jumps to the selected device, or the configured default lat/lon/zoom, or fits bounds around all visible positions. Fires only once (guarded by `initialized` state).
**Fits in:** Rendered by `MainMap.jsx`. Pure behavior component; returns `null`. Uses Redux for selected device and positions; uses preferences for default coordinates.
**Read next:** [[MapSelectedDevice.js]] (handles subsequent camera changes after init), [[MapCamera.js]] (generic fit-bounds helper used elsewhere).

## Public API
- `MapDefaultCamera({ filteredPositions })` (line 7, default export) — `filteredPositions?: Position[]` — when provided, used instead of all positions for the auto-fit bounds fallback. Returns `null`.

## Key flows

### Priority order (lines 17-68)
Single `useEffect` with `initialized` guard:
1. **Selected device** (lines 19-26): if `selectedDeviceId` is set and its position exists → `map.jumpTo(center, zoom ≥ selectZoom ≥ 10)`.
2. **Configured default** (lines 28-34): if `defaultLatitude` and `defaultLongitude` preferences are set → `map.jumpTo(center, defaultZoom)`.
3. **Auto-fit multiple positions** (lines 35-48): `filteredPositions ?? Object.values(positions)` → `LngLatBounds.reduce` → `map.fitBounds(bounds, { duration: 0, padding: 10% })`.
4. **Single position** (lines 49-56): if exactly one coordinate → `map.jumpTo`.
5. If none of the above match (e.g. `selectedDeviceId` set but position not yet loaded) → `initialized` stays false, effect reruns on next dep change.

### `initialized` latch
Once `setInitialized(true)` is called in any branch, the effect exits immediately on subsequent runs (line 18). This prevents the camera from jumping after the user has manually panned.

## Gotchas / non-obvious
- **`defaultZoom`** preference defaults to `0` (line 13) — `map.jumpTo` to zoom 0 shows the whole earth. If server has no zoom configured, the auto-fit branch handles it instead (because `defaultLatitude`/`defaultLongitude` would also be falsy).
- **`filteredPositions` prop** is passed from `MainMap.jsx` as the device-list-filtered subset — so on a filtered view, the initial camera fits only visible devices.
- **Effect deps include `filteredPositions`** — if the filtered list arrives asynchronously after first render, the effect reruns and initializes then.

## Line index
- 7 — component declaration
- 15 — `initialized` state
- 17 — effect with `initialized` early-return
- 19-26 — branch 1: selected device
- 28-34 — branch 2: configured default
- 35-48 — branch 3: auto-fit multiple
- 49-56 — branch 4: single position
