# `src/map/overlay/` — Raster tile overlays

Two files: a hook that defines 12 raster overlay providers, and the component that renders the selected one.

## File index

| File | One-liner | Annotation |
|---|---|---|
| `useMapOverlays.js` | 12 raster overlay descriptors (traffic, weather, sea, rail, custom) | [useMapOverlays.js.md](useMapOverlays.js.md) |
| `MapOverlay.js` | Finds the active overlay and adds it as a raster source+layer | [MapOverlay.js.md](MapOverlay.js.md) |

## Architecture

Mirrors the `useMapStyles.js` / `MapView.jsx` relationship:
- `useMapOverlays.js` is the data layer (descriptors, API keys, availability).
- `MapOverlay.js` is the rendering layer (one source + one `'raster'` layer on the map).

Unlike the base styles (which are full GL style JSONs replacing the entire map style), overlays are raster tile layers **added on top** of the base style. Only one overlay is active at a time.

## Provider list

| ID | Key required | Free? |
|---|---|---|
| `googleTraffic` | `googleKey` | No |
| `openSeaMap` | — | Yes |
| `openRailwayMap` | — | Yes |
| `openWeatherClouds` | `openWeatherKey` | No |
| `openWeatherPrecipitation` | `openWeatherKey` | No |
| `openWeatherPressure` | `openWeatherKey` | No |
| `openWeatherWind` | `openWeatherKey` | No |
| `openWeatherTemperature` | `openWeatherKey` | No |
| `tomTomFlow` | `tomTomKey` | No |
| `tomTomIncidents` | `tomTomKey` | No |
| `hereFlow` | `hereKey` | No |
| `custom` | — | Depends on `overlayUrl` server attribute |

## Adding a new overlay

1. Add an entry to `useMapOverlays.js` array.
2. If key-gated: add `useAttributePreference('yourKey')` and add to `useMemo` deps.
3. Use `sourceCustom(urls, maxZoom)` helper for raster tile URLs.
4. No changes needed to `MapOverlay.js`.
5. The overlay will appear in whatever UI reads the overlay list (currently `src/settings/ServerPage.jsx`'s overlay selector).
