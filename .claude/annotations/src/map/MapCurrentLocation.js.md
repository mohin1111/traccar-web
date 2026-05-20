# MapCurrentLocation.js

**Role:** Adds the built-in MapLibre `GeolocateControl` (browser geolocation dot) to the map control bar. One-shot mode — locates but does not continuously track.
**Fits in:** Rendered by `MainMap.jsx`. Trivial wrapper around a MapLibre built-in control.
**Read next:** [[MapView.jsx]] (map singleton).

## Public API
- `MapCurrentLocation()` (line 6, default export) — no props. Returns `null`.

## Key flows
Single `useEffect` on `theme.direction`: creates `maplibregl.GeolocateControl({ enableHighAccuracy: true, timeout: 5000, trackUserLocation: false })`, adds to `top-right` (or `top-left` for RTL), returns cleanup that removes it.

## Gotchas / non-obvious
- **`trackUserLocation: false`** — fires a single locate-and-center, does not continuously update the blue dot as the user moves. The dot is the browser's own location indicator, not a tracked device.
- **5 second timeout** — if the browser takes longer to get a GPS fix, the control shows an error state.
- Control is removed and recreated when `theme.direction` changes (for RTL support).

## Line index
- 6 — component declaration
- 9-18 — GeolocateControl add effect
