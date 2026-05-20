# mapUtil.js

**Role:** Pure utility module — image tinting canvas ops, geofence WKT↔GeoJSON conversion, coordinate array reversal (lat/lng ↔ lng/lat), and font family detection for the active tile provider.
**Fits in:** Imported by `preloadImages.js` (`loadImage`, `prepareIcon`), `MapGeofence.js`, `MapGeofenceEdit.js`, `MapRuler.jsx`, `MapRouteCoordinates.js`, `MapRoutePoints.jsx`, `PoiMap.js`, `MapMarkers.js`. No React; no Redux.
**Read next:** [[preloadImages.js]] (uses `loadImage` + `prepareIcon`), [[MapGeofence.js]] (uses `geofenceToFeature`), [[MapGeofenceEdit.js]] (uses `geofenceToFeature` + `geometryToArea`).

## Public API
- `loadImage(url)` (lines 4-9) — returns `Promise<HTMLImageElement>`.
- `prepareIcon(background, icon?, color?)` (lines 32-56) — composites icon onto background on a HiDPI canvas; returns `ImageData` for `map.addImage()`. Tinting via `destination-atop` composite op.
- `reverseCoordinates(it)` (lines 58-72) — recursively swaps `[lat, lng]` ↔ `[lng, lat]`. WKT/wellknown parser emits `[lat, lng]`; MapLibre expects `[lng, lat]`.
- `geofenceToFeature(theme, item)` (lines 74-102) — converts a Traccar geofence REST object to a GeoJSON Feature. Handles CIRCLE (converted to polygon via `@turf/circle`) and all WKT geometry types (via `wellknown` parse).
- `geometryToArea(geometry)` (lines 104) — inverse: GeoJSON geometry → WKT string (Traccar REST format for `area` field).
- `findFonts(map)` (lines 106-112) — inspects `map.getStyle().glyphs` URL to return the correct font stack. OpenFreeMap uses `['Noto Sans Regular']`; all others use `['Open Sans Regular', 'Arial Unicode MS Regular']`.

## Key flows

### `prepareIcon` canvas tinting (lines 11-56)
`canvasTintImage(image, color)`: fills canvas with `color`, then composites image on top with `destination-atop` blend — effectively colorizes the transparent-background SVG to a solid color. `prepareIcon` draws the background first, then overlays the tinted icon at 50% size centered.

### CIRCLE geofence expansion (lines 76-88)
Traccar stores circles as `"CIRCLE (lat lon, radius)"`. The parser strips the WKT-like string with a regex, extracts center `[lon, lat]` and radius in meters, and calls `turfCircle` with `steps: 32` to produce a 32-vertex polygon. This is needed because MapLibre has no native circle geometry type.

### `reverseCoordinates` recursion (lines 58-72)
Three cases: `null` passthrough, `[number, number]` leaf swap, array of arrays recursive map, GeoJSON object (spreads and recursively processes `.coordinates`). This handles Points, LineStrings, Polygons, and MultiPolygons uniformly.

## Gotchas / non-obvious
- **`wellknown` library** (`parse`/`stringify`) handles WKT → GeoJSON and back. It emits `[lat, lng]` order — hence `reverseCoordinates` is always applied on parse output before use in MapLibre.
- **`findFonts` inspects live style** — must be called after the map style is loaded. Called inside effects that run when `mapReady` is true, so this is safe.
- **`@turf/circle` radius is in meters** but the options say `units: 'meters'` — this is correct. Turf defaults to kilometers so the explicit unit is required.
- **`prepareIcon` returns `ImageData`** (from `context.getImageData`), not a canvas or element. `map.addImage(key, imageData, { pixelRatio })` accepts this directly.

## Line index
- 4-9 — `loadImage`
- 11-30 — `canvasTintImage` (destination-atop tint)
- 32-56 — `prepareIcon` (background + centered icon composite)
- 58-72 — `reverseCoordinates`
- 74-102 — `geofenceToFeature` (CIRCLE branch + WKT branch)
- 104 — `geometryToArea`
- 106-112 — `findFonts` (OpenFreeMap detection)
