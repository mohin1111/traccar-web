# MapGeocoder.jsx

**Role:** Search-by-address control that adds a magnifying-glass button to the MapLibre control bar; on click opens a MUI Popover with a debounced Nominatim text search and fits the map to the selected result's bounding box.
**Fits in:** Rendered by `MainMap.jsx`. Injects itself into the map control group via `map.addControl()`. Dispatches to `errorsActions` on fetch failure.
**Read next:** [[MapView.jsx]] (provides `map` singleton), [[MapSwitcher.jsx]] (same control-injection pattern).

## Public API
- `MapGeocoder` (default export, line 43) — no props. Fully self-contained. Mounts a MapLibre control button via React portal (`createRoot`).

## Key flows

### Control injection (lines 80-103)
Uses the MapLibre custom control interface (`onAdd` returns a DOM element, `onRemove` cleans up). The button element hosts a React root via `createRoot` rendering a MUI `TravelExploreIcon`. `button.onclick` sets `anchorEl` state to open the Popover. Control is placed `top-right` (LTR) or `top-left` (RTL).

### Search with debounce + abort (lines 53-78)
`useEffect` on `query` change: 300 ms `setTimeout` debounce; creates `AbortController` to cancel in-flight requests on each keystroke. Calls Nominatim `search?format=geojson&addressdetails=1`. On success sets `results` to `data.features`. AbortError is silently swallowed; other errors push to Redux error queue.

### Map fit on selection (lines 105-117)
`onSelect(feature)` calls `map.fitBounds([[minX, minY], [maxX, maxY]], { padding: 40 })`. Clears query and results. Closes Popover.

## Gotchas / non-obvious
- **`queueMicrotask(() => iconRoot.unmount())`** (line 97) — defers React root unmount to the next microtask to avoid "unmount during render" warnings when MapLibre calls `onRemove` synchronously.
- **Nominatim ToS** requires a `User-Agent` header in production. The current implementation sends no custom header — acceptable for the demo/self-hosted context but may be rate-limited by public Nominatim.
- **`anchorEl` is set to the DOM button element** (not an event) — MUI Popover uses it as the anchor for `anchorOrigin` calculation.

## Line index
- 43 — component declaration
- 53-78 — debounced search effect
- 80-103 — control injection (onAdd/onRemove + createRoot)
- 105-117 — `onSelect` fit-to-bounds
- 119-153 — JSX: Popover + TextField + results List
