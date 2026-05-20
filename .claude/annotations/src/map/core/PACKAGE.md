# `src/map/core/` — Map foundation

Four files that everything else in `src/map/` depends on. Contains the global singleton, the tile provider registry, pure utilities, and the image sprite system.

## File index

| File | One-liner | Annotation |
|---|---|---|
| `MapView.jsx` | Global `maplibregl.Map` singleton; style switching; ready-state pub/sub; mounts into `containerRef` | [MapView.jsx.md](MapView.jsx.md) |
| `useMapStyles.js` | Hook returning 26 tile/style descriptors; drives `MapSwitcher` and style selection | [useMapStyles.js.md](useMapStyles.js.md) |
| `mapUtil.js` | Icon canvas compositing, WKT↔GeoJSON, coordinate reversal, font detection | [mapUtil.js.md](mapUtil.js.md) |
| `preloadImages.js` | Async sprite loader: 22 icons × 4 colors → `mapImages`; `mapIconKey` normalizer | [preloadImages.js.md](preloadImages.js.md) |

## Dependency graph

```
MapView.jsx
 ├── useMapStyles.js   (called inside MapView component)
 ├── preloadImages.js  (mapImages registered via initMap())
 └── MapSwitcher.jsx   (rendered inside MapView JSX)

mapUtil.js      ← imported by preloadImages.js, MapGeofence.js,
                   MapGeofenceEdit.js, MapRuler.jsx, MapRouteCoordinates.js,
                   MapRoutePoints.jsx, PoiMap.js, MapMarkers.js

preloadImages.js ← depends on mapUtil.js (loadImage, prepareIcon)
                 ← imported by MapView.jsx (initMap), MapPositions.js (mapIconKey)
```

## Key invariants

- `map` singleton is created at module-evaluate time — before any React component mounts.
- `mapImages` dict is populated by the async default export of `preloadImages.js`, then entries are registered via `map.addImage()` in `initMap()` inside `MapView.jsx`. This sequence must complete before children render.
- `findFonts(map)` must be called after a style is loaded (always safe inside `useEffect` on `mapReady` children).
- All 26 providers in `useMapStyles.js` share the `styleCustom` helper for raster-only providers; vector providers use a bare URL string.

## External dependencies (npm)

- `maplibre-gl` — the core library
- `maplibre-google-maps` — `googleProtocol` for `google://` tile URLs
- `@turf/circle` — CIRCLE geofence expansion in `mapUtil.js`
- `wellknown` — WKT parse/stringify in `mapUtil.js`
- `@mui/material` — theme (color tokens), `useTheme` in `MapView.jsx`
- `react-redux` — `useSelector` in `useMapStyles.js` (custom map URL from server state)
