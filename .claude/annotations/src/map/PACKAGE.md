# `src/map/` — MapLibre map subsystem

React wrapper layer around MapLibre GL JS for live tracking, replay, geofence editing, and map overlays. The most reusable and vendor-able part of the Traccar web frontend.

## File index

| File | One-liner | Annotation |
|---|---|---|
| `core/MapView.jsx` | Global map singleton; style switching; sprite preloading; ready pub/sub | [MapView.jsx.md](core/MapView.jsx.md) |
| `core/useMapStyles.js` | 26-provider tile/style registry (OpenFreeMap → Mapbox → Custom) | [useMapStyles.js.md](core/useMapStyles.js.md) |
| `core/mapUtil.js` | Icon compositing, geofence WKT↔GeoJSON, coordinate reversal, font detection | [mapUtil.js.md](core/mapUtil.js.md) |
| `core/preloadImages.js` | SVG sprite preloader: 22 icons × 4 colors + background + direction | [preloadImages.js.md](core/preloadImages.js.md) |
| `control/MapGeocoder.jsx` | Nominatim address search in MapLibre control bar | [MapGeocoder.jsx.md](control/MapGeocoder.jsx.md) |
| `control/MapNotification.jsx` | Events-drawer bell button (toggles red when active) | [MapNotification.jsx.md](control/MapNotification.jsx.md) |
| `control/MapRuler.jsx` | Click-to-measure distance ruler with snap-to-position | [MapRuler.jsx.md](control/MapRuler.jsx.md) |
| `control/MapSpeedLegend.jsx` | Turbo-colormap speed legend bar (replay pages) | [MapSpeedLegend.jsx.md](control/MapSpeedLegend.jsx.md) |
| `control/MapSwitcher.jsx` | Layer-picker menu (base tile style switcher) | [MapSwitcher.jsx.md](control/MapSwitcher.jsx.md) |
| `draw/MapGeofenceEdit.js` | MapboxDraw-based geofence CRUD editor (polygon/line/trash) | [MapGeofenceEdit.js.md](draw/MapGeofenceEdit.js.md) |
| `draw/theme.js` | MapboxDraw default theme styles (static copy of v1.4.0) | [theme.js.md](draw/theme.js.md) |
| `main/MapAccuracy.js` | GPS accuracy halos (semi-transparent circles) | [MapAccuracy.js.md](main/MapAccuracy.js.md) |
| `main/MapDefaultCamera.js` | One-time initial camera: selected device → default coords → auto-fit | [MapDefaultCamera.js.md](main/MapDefaultCamera.js.md) |
| `main/MapLiveRoutes.js` | Breadcrumb polylines from session.history (none/selected/all modes) | [MapLiveRoutes.js.md](main/MapLiveRoutes.js.md) |
| `main/MapSelectedDevice.js` | Camera follow: easeTo on device change or position update | [MapSelectedDevice.js.md](main/MapSelectedDevice.js.md) |
| `main/PoiMap.js` | KML point-of-interest layer (fetched from poiLayer preference URL) | [PoiMap.js.md](main/PoiMap.js.md) |
| `overlay/MapOverlay.js` | Single raster overlay layer renderer | [MapOverlay.js.md](overlay/MapOverlay.js.md) |
| `overlay/useMapOverlays.js` | 12 raster overlay descriptors (traffic, weather, sea, rail, custom) | [useMapOverlays.js.md](overlay/useMapOverlays.js.md) |
| `MapCamera.js` | Generic fit-bounds / jumpTo helper (report + replay pages) | [MapCamera.js.md](MapCamera.js.md) |
| `MapCurrentLocation.js` | Browser GeolocateControl wrapper (one-shot browser location) | [MapCurrentLocation.js.md](MapCurrentLocation.js.md) |
| `MapGeofence.js` | Read-only geofence fill+line+title display layer | [MapGeofence.js.md](MapGeofence.js.md) |
| `MapMarkers.js` | Static symbol marker layer (route start/end, generic icons) | [MapMarkers.js.md](MapMarkers.js.md) |
| `MapPadding.js` | Side-panel-aware map padding (shifts controls + fitBounds area) | [MapPadding.js.md](MapPadding.js.md) |
| `MapPositions.js` | Core live-tracking layer: clustered markers, status colors, direction | [MapPositions.js.md](MapPositions.js.md) |
| `MapRouteCoordinates.js` | Named solid-color polyline for report views (with title label) | [MapRouteCoordinates.js.md](MapRouteCoordinates.js.md) |
| `MapRoutePath.js` | Speed-color-coded replay path (per-segment Turbo gradient) | [MapRoutePath.js.md](MapRoutePath.js.md) |
| `MapRoutePoints.jsx` | Clickable speed-colored route arrows (▲) + optional speed legend | [MapRoutePoints.jsx.md](MapRoutePoints.jsx.md) |
| `MapScale.js` | Unit-aware scale bar (metric/imperial/nautical) | [MapScale.js.md](MapScale.js.md) |

## The global `map` singleton pattern

**This is the most important design decision in the subsystem.**

```js
// core/MapView.jsx — module-level, runs once
export const map = new maplibregl.Map({ container: element, attributionControl: false });
```

Every other file in `src/map/` imports this named export:
```js
import { map } from '../core/MapView';  // or '../../map/core/MapView'
```

All map mutations happen imperatively inside React `useEffect` hooks:
```js
useEffect(() => {
  map.addSource(id, { type: 'geojson', data: emptyFeatureCollection });
  map.addLayer({ id, type: 'symbol', source: id, ... });
  return () => {
    if (map.getLayer(id)) map.removeLayer(id);
    if (map.getSource(id)) map.removeSource(id);
  };
}, [deps]);
```

**Children are gated behind `{mapReady && children}` in `MapView.jsx`.** When a style switch occurs, `mapReady` goes `false → true`, unmounting and remounting all children. Every child re-runs its setup effect. This is why every child must include a cleanup function.

## `addSource` / `addLayer` / `setData` lifecycle

```
mapReady = false  (style loading)
  ↓
mapReady = true   (style loaded, images registered)
  ↓
Child mounts → useEffect fires → map.addSource(id, emptyData)
                                → map.addLayer(...)
  ↓
Data arrives → second useEffect fires → map.getSource(id)?.setData(newData)
  ↓
Data updates → same second effect → setData again (in-place, no layer re-creation)
  ↓
Style change → mapReady = false → children unmount → cleanup removes layers+sources
            → mapReady = true → children remount → start over
```

**Critical invariants:**
- `map.addSource` must precede `map.addLayer` referencing that source.
- Defensive `map.getLayer(id)` / `map.getSource(id)` checks before remove (style swap can remove them externally).
- `map.getSource(id)?.setData(...)` uses optional chaining — safe to call before source exists (returns undefined).
- Source IDs from `useId()` are React-stable unique strings; hardcoded IDs are used only for singleton layers.

## 26 tile providers in `useMapStyles.js`

Free (always `available: true`): OpenFreeMap, LocationIQ Streets, LocationIQ Dark, OSM, OpenTopoMap, Carto, Google Road/Satellite/Hybrid (unauthenticated CDN), Yandex, AutoNavi, Ordnance Survey.

Key-gated (`available: Boolean(key)`): MapTiler Basic/Hybrid (`mapTilerKey`), Bing Road/Aerial/Hybrid (`bingMapsKey`), TomTom (`tomTomKey`), HERE Basic/Hybrid/Satellite (`hereKey`), Mapbox Streets/Dark/Outdoors/Satellite (`mapboxAccessToken`), Custom URL (`mapUrl` server attribute).

**For Transport OS:** MapmyIndia/Ola Maps slot in via the Custom URL provider — set `mapUrl` to their style JSON endpoint.

## 12 overlay providers in `useMapOverlays.js`

Google Traffic (`googleKey`), OpenSeaMap (free), OpenRailwayMap (free), OpenWeather ×5 (`openWeatherKey`), TomTom Flow+Incidents (`tomTomKey`), HERE Flow (`hereKey`), Custom URL (`overlayUrl` server attribute).

## What to copy together to vendor this subsystem

**Minimum required files from outside `src/map/`:**

| File | Why needed |
|---|---|
| `src/resources/images/` | All SVG marker icons + background + direction |
| `src/common/util/preferences.js` | `useAttributePreference`, `usePreference` hooks |
| `src/common/util/usePersistedState.js` | `selectedMapStyle` persistence |
| `src/common/util/colors.js` | `getSpeedColor`, `interpolateTurbo` |
| `src/common/util/formatter.js` | `formatTime`, `formatDistance`, `getStatusColor` |
| `src/common/util/converter.js` | `speedFromKnots`, `speedUnitString` |
| `src/common/util/fetchOrThrow.js` | Used by `MapGeofenceEdit.js` |
| `src/reactHelper.js` | `useAsyncTask`, `useCatchCallback`, `usePrevious` |
| `src/common/components/LocalizationProvider.jsx` | `useTranslation` |
| `src/common/theme/dimensions.js` | `popupMapOffset` constant |
| Redux slices: `session`, `devices`, `geofences`, `errors` | Live data + error dispatch |

**npm packages required:**
- `maplibre-gl` (core)
- `maplibre-google-maps` (google:// protocol)
- `@mapbox/mapbox-gl-draw` + CSS (geofence editing only)
- `@turf/circle` (CIRCLE geofences + accuracy halos)
- `wellknown` (WKT parse/stringify)
- `@tmcw/togeojson` (KML → GeoJSON, PoiMap only)
- `tss-react` + MUI (theming + controls)
- `react-redux` (data binding)
- `react-router-dom` (navigate in MapGeofenceEdit)

**If only vendoring the live tracking view** (not geofence editing, not KML POI, not reports): can drop `MapGeofenceEdit.js`, `theme.js`, `PoiMap.js`, `MapRouteCoordinates.js`, `MapRoutePath.js`, `MapRoutePoints.jsx`, and associated `@mapbox/mapbox-gl-draw`, `@tmcw/togeojson`, `wellknown`.

## Dependency graph (intra-subsystem)

```
MapView.jsx
 ├── useMapStyles.js  (tile styles)
 ├── MapSwitcher.jsx  (UI inside MapView)
 └── preloadImages.js → mapUtil.js

MapPositions.js → MapView.jsx, preloadImages.js, mapUtil.js
MapGeofence.js  → MapView.jsx, mapUtil.js
MapGeofenceEdit.js → MapView.jsx, mapUtil.js, theme.js
MapLiveRoutes.js   → MapView.jsx
MapAccuracy.js     → MapView.jsx
MapOverlay.js      → MapView.jsx, useMapOverlays.js
MapRoutePath.js    → MapView.jsx
MapRouteCoordinates.js → MapView.jsx, mapUtil.js
MapRoutePoints.jsx → MapView.jsx, MapSpeedLegend.jsx, mapUtil.js
MapRuler.jsx   → MapView.jsx, mapUtil.js
PoiMap.js      → MapView.jsx, mapUtil.js
```

## Multi-file flows

### Live tracking (main page)
1. `SocketController` dispatches `{positions}` → `sessionActions.updatePositions`.
2. `MainMap.jsx` passes `Object.values(state.session.positions)` as `positions` prop to `MapPositions`.
3. `MapPositions` second effect fires → `map.getSource(id).setData(...)` — in-place update, no React re-render of layers.
4. `MapLiveRoutes` reads `state.session.history` → `setData` on its polyline source.
5. `MapSelectedDevice` watches `devices.selectedId` + `devices.selectTime` → `map.easeTo`.

### Replay
1. `ReplayPage` fetches positions, passes to `MapPositions`, `MapRoutePath`, `MapRoutePoints`, `MapAccuracy`, `MapRuler`.
2. User scrubs → `MapCamera` fires `fitBounds`; clicking route point fires `onClick(id, index)` → scrubber seeks.

### Geofence editing
1. `GeofencesPage` renders `MapGeofenceEdit` + `MapGeofence`.
2. `MapGeofenceEdit` loads geofences, `draw.add()`s them, listens for `draw.create`/`draw.update`/`draw.delete`.
3. REST CRUD fires on each draw event → Redux `geofencesActions.refresh` → `MapGeofence` setData updates.

## Conventions

- **`useId()` for source IDs** (all recent files) — prevents collision when multiple instances of the same component exist on one page.
- **Hardcoded layer IDs** for singleton layers (e.g. `'geofences-fill'`, `'poi-fill'`) — acceptable since those components are always singletons.
- **`map.getLayer(id)` guard before `removeLayer`** — MapLibre throws if you remove a layer that doesn't exist (e.g. after a style swap removed it first).
- **`queueMicrotask(() => root.unmount())`** — defers React root unmount to avoid "unmount during render" warning in all `control/` files.
- **Ref forwarding for stable callbacks** — `onClickRef`, `positionsRef`, `onActiveChangeRef` pattern used in `MapRuler` and `MapNotification` to avoid re-registering MapLibre event listeners on every render.
- **`usePrevious`** from `reactHelper.js` for change detection (MapSelectedDevice).

## How to add a new map layer

1. Create a new file in `src/map/` (or a subdirectory).
2. Import `{ map }` from `../core/MapView`.
3. Use `useId()` for source/layer IDs.
4. First `useEffect`: `map.addSource(id, {...})` + `map.addLayer(...)` + cleanup.
5. Second `useEffect`: `map.getSource(id)?.setData(...)` on data prop change.
6. Return `null`.
7. Render the component inside `MapView` children (so it mounts only after `mapReady`).

## How to add a new tile provider

1. Add an entry to the array in `useMapStyles.js`.
2. If key-gated: add `useAttributePreference('yourKey')` near the top and add to `useMemo` deps.
3. If raster-only: use `styleCustom({ tiles: [...], maxZoom: N })`.
4. If vector: provide a style JSON URL directly as `style`.
5. Add the `id` string to the `activeMapStyles` attribute in the Traccar server settings UI to make it appear in the switcher by default.

## How to add a new raster overlay

1. Add an entry to the array in `useMapOverlays.js`.
2. Use `sourceCustom(urls, maxZoom)` helper.
3. Add any API key with `useAttributePreference('yourKey')` and add to `useMemo` deps.
4. The `MapOverlay.js` consumer requires no changes.
