# MapView.jsx

**Role:** Owns the single global `maplibregl.Map` instance and its DOM container; handles style switching, image preloading, RTL text plugin, and ready-state propagation to all map children via a subscription model.
**Fits in:** The root of every page that renders a map. `map` and the ready system are the dependency hub for every other file in `src/map/`. Rendered by `MainMap.jsx`, `ReplayPage.jsx`, `GeofencesPage.jsx`, and others.
**Read next:** [[MapPositions.js]] (primary consumer of the singleton), [[useMapStyles.js]] (provides the 26 style objects), [[preloadImages.js]] (populates `mapImages` dict).

## Public API
- `map` (line 20) — **the global MapLibre singleton**. Exported named. Every other `src/map/` file imports and mutates this object directly via `map.addSource`/`map.addLayer`.
- `MapView` (line 53, default export) — React component. Accepts `children`; renders children only after `mapReady === true` (line 146). Mounts the detached `element` div into its `containerRef`.
- `addReadyListener` / `removeReadyListener` (lines 28-35) — module-private ready pub/sub (not exported; children subscribe indirectly via rendering under `MapView`).

## Key flows

### Singleton initialization (module load, lines 13-51)
The `Map` is constructed once at module evaluation time on a detached `element` div (line 13-23). This means `map` exists before any React component mounts. `maplibregl.addProtocol('google', googleProtocol)` also runs once at module load (line 18).

### Style switching (lines 104-124)
1. `selectedStyleId` changes → `updateReadyValue(false)` (blocks children from rendering).
2. `map.setStyle(style.style, { diff: false })` — full style swap, not incremental.
3. `map.once('styledata', waiting)` polls `map.loaded()` on a 33 ms timer.
4. When loaded: `initMap()` (re-adds `mapImages` sprites) → `updateReadyValue(true)`.
5. Children re-render. All `useEffect` add-source/add-layer hooks re-run.

**Critical:** Every style swap resets all sources and layers. Children must add their sources/layers fresh each time `mapReady` becomes true (which they do because `{mapReady && children}` re-mounts them).

### DOM attachment (lines 134-141)
`useLayoutEffect` appends the persistent `element` into `containerRef.current` and calls `map.resize()`. On unmount it removes the element — the `Map` object stays alive, ready to be reattached elsewhere.

### RTL plugin (lines 77-81)
`maplibregl.setRTLTextPlugin('/mapbox-gl-rtl-text.js')` — called once when `theme.direction === 'rtl'`. The file must be present at the build root (copied via `vite-plugin-static-copy`).

## Gotchas / non-obvious
- **Map is created before React renders.** Accessing `map` outside a `mapReady` guard will work for method calls that don't require a loaded style, but adding sources/layers before style is loaded throws.
- **`{ diff: false }` on setStyle** means MapLibre does NOT preserve existing sources/layers across style changes — all children must re-add them after each ready cycle.
- **`mapImages` is populated imperatively** inside `initMap()` (line 44-50), which re-runs after every style switch to restore sprites. The `preloadImages` default export is called separately at app startup.
- **`mapboxAccessToken` is set globally** on `maplibregl.accessToken` (line 101) even when Mapbox styles are not selected — harmless but affects any Mapbox SDK internals that inspect the global.
- **`mapReady && children`** means children are completely unmounted and remounted on style change, not just re-rendered. All their `useEffect` teardowns and setups run.

## Line index
- 13-17 — detached DOM element construction
- 18 — Google Maps protocol registration
- 20-23 — `map` singleton construction (the most important line in the file)
- 25-40 — ready pub/sub (module-private)
- 42-51 — `initMap()` re-registers sprites after style change
- 60-68 — style selection from preferences + persisted state
- 77-81 — RTL plugin registration
- 83-92 — attribution + navigation controls
- 104-124 — style-switch + ready lifecycle
- 134-141 — DOM mount/unmount via `useLayoutEffect`
- 143-148 — JSX: renders `MapSwitcher` + `{mapReady && children}`
