# MapSelectedDevice.js

**Role:** Smoothly pans/zooms the map (`map.easeTo`) whenever the selected device changes, the select action is re-fired (e.g. double-click same device), or — if `mapFollow` is enabled — whenever the selected device's position updates.
**Fits in:** Rendered by `MainMap.jsx`. Reads from `devices` Redux slice (`selectedId`, `selectTime`) and `session.positions`. Returns `null`.
**Read next:** [[MapDefaultCamera.js]] (one-time initial camera), [[MapCamera.js]] (imperative fit-bounds used for multi-position views).

## Public API
- `MapSelectedDevice()` (line 8, default export) — no props. Returns `null`.

## Key flows

### Change detection (lines 21-37)
`usePrevious` hook (from `reactHelper`) captures previous values of `currentId`, `currentTime`, and `position`. The effect fires when:
- Device selection changed (`currentId !== previousId`) — different device selected.
- Same device re-selected (`currentTime !== previousTime`) — `selectTime` is a timestamp updated on each click, even same-device.
- Position moved (`mapFollow && positionChanged`) — follow mode keeps map centered.

Camera: `map.easeTo({ center: [lon, lat], zoom: max(current, selectZoom), offset: [0, -popupMapOffset/2] })`. The negative Y offset compensates for the info popup that appears below center.

### `mapFollow` behavior
`useAttributePreference('mapFollow', false)` — when true, any position update for the selected device triggers `easeTo`. This can be disorienting if the user is panning manually; Traccar leaves it opt-in.

## Gotchas / non-obvious
- **`selectZoom` default is 10** — minimum zoom when selecting. `Math.max(map.getZoom(), selectZoom)` ensures we never zoom OUT when selecting a device.
- **`dimensions.popupMapOffset`** is imported from the theme dimensions object — a pixel constant for the popup height, halved for centering.
- **`usePrevious`** is from `reactHelper.js`, not a standard hook — returns the value from the previous render via a ref.
- **`selectTime` is in Redux** — updated by `devicesActions.select`. Without it, clicking the same device twice would not re-fire the camera (since `currentId` wouldn't change).

## Line index
- 9-11 — currentId, selectTime, previousTime, previousId via usePrevious
- 14-15 — selectZoom, mapFollow preferences
- 17-19 — position + previousPosition
- 21-39 — effect: change detection + easeTo
- 26-29 — positionChanged computation
- 30-37 — three trigger conditions (or-ed)
