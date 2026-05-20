# `src/map/draw/` — Geofence drawing tools

Two files that implement the geofence editing UI using `@mapbox/mapbox-gl-draw` adapted for MapLibre.

## File index

| File | One-liner | Annotation |
|---|---|---|
| `MapGeofenceEdit.js` | Full CRUD geofence editor: MapboxDraw + REST API + Redux + navigation | [MapGeofenceEdit.js.md](MapGeofenceEdit.js.md) |
| `theme.js` | Static draw style array (copy of mapbox-gl-draw v1.4.0 default theme) | [theme.js.md](theme.js.md) |

## MapboxDraw / MapLibre compatibility

`@mapbox/mapbox-gl-draw` targets Mapbox GL JS. To use it with MapLibre, three internal CSS class name constants are patched at module load (lines 17-19 of `MapGeofenceEdit.js`):

```js
MapboxDraw.constants.classes.CONTROL_BASE   = 'maplibregl-ctrl';
MapboxDraw.constants.classes.CONTROL_PREFIX = 'maplibregl-ctrl-';
MapboxDraw.constants.classes.CONTROL_GROUP  = 'maplibregl-ctrl-group';
```

Additionally, `@mapbox/mapbox-gl-draw/dist/mapbox-gl-draw.css` is imported — this stylesheet uses `.mapboxgl-*` class names internally for draw geometries (vertex handles, midpoints). The patch above only fixes the control bar button; the draw-canvas CSS may diverge if MapboxDraw CSS uses `.mapboxgl-` class names for draw handles.

## REST API endpoints used

| Action | Method | Endpoint |
|---|---|---|
| Load geofences | GET | `/api/geofences` |
| Create geofence | POST | `/api/geofences` |
| Update geofence | PUT | `/api/geofences/:id` |
| Delete geofence | DELETE | `/api/geofences/:id` |

## Who renders this

`GeofencesPage.jsx` (from `src/other/`) renders `MapGeofenceEdit` + `MapGeofence` together. `MapGeofence` (read-only) shows existing geofences as a static layer; `MapGeofenceEdit` loads them into the draw state so they become interactive.

## External dependencies (npm)

- `@mapbox/mapbox-gl-draw` (v1.4.x) + its CSS
- `react-router-dom` (`useNavigate` for post-create redirect)
- `react-redux` (geofences slice + errorsActions)
- `../../reactHelper` (`useCatchCallback`)
- `../../common/util/fetchOrThrow`
