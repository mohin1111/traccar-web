# preloadImages.js

**Role:** Defines the 22-icon SVG sprite set, exports `mapIcons` (name→SVG URL), `mapIconKey` (category normalizer), and `mapImages` (the live image dict populated by the async default export). The async default function pre-renders all icon+color combinations onto HiDPI canvases and populates `mapImages` before the map is ready.
**Fits in:** Called once at app startup (or on each style switch via `initMap()` in `MapView.jsx`). `mapImages` is consumed by `MapView.jsx:initMap()` which registers each entry via `map.addImage()`. `mapIconKey` is consumed by `MapPositions.js` and `MapMarkers.js`.
**Read next:** [[mapUtil.js]] (`loadImage`, `prepareIcon`), [[MapView.jsx]] (`initMap` registers the images), [[MapPositions.js]] (uses `mapIconKey`).

## Public API
- `mapIcons` (lines 30-53) — `Record<string, string>` mapping icon name to SVG module URL. 22 icons: animal, bicycle, boat, bus, car, camper, crane, default, finish, helicopter, motorcycle, person, plane, scooter, ship, start, tractor, trailer, train, tram, truck, van.
- `mapIconKey(category)` (lines 55-65) — normalizes a Traccar device `category` string to a key in `mapIcons`. `offroad`/`pickup` → `'car'`; `trolleybus` → `'bus'`; unknown → `'default'`.
- `mapImages` (line 67) — mutable `{}` object. After the async default export runs, keys follow the pattern `'<icon>-<color>'` (e.g. `'car-success'`, `'truck-error'`) plus `'background'` and `'direction'`. Exported by reference — callers read the same object.
- Default export (lines 75-96) — async function. Loads and composites all images; populates `mapImages` in place.

## Key flows

### Image generation (lines 75-96)
1. Load `backgroundSvg` once.
2. `mapImages.background` = background without icon (used for cluster circles).
3. `mapImages.direction` = direction arrow SVG without tint.
4. For each of 22 icons × 4 colors (`info`, `success`, `error`, `neutral`): load icon SVG, tint it with `theme.palette[color].main`, composite onto background → store as `mapImages[${category}-${color}]`.
5. All loads are parallelized via `Promise.all`.

### Color scheme (lines 82-88)
A minimal `createTheme` with `neutral: grey[500]` is created locally (line 69-73) solely to resolve MUI color tokens to hex strings for canvas tinting. This is the only MUI usage in this file.

### Naming convention
MapLibre image keys: `'background'`, `'direction'`, `'car-info'`, `'car-success'`, `'car-error'`, `'car-neutral'`, etc. `MapPositions.js` generates `'{category}-{color}'` expressions that reference these exact keys.

## Gotchas / non-obvious
- **`mapImages` is shared mutable state.** The async export mutates it; `MapView.jsx:initMap()` then calls `map.addImage()` for each key. If `initMap` runs before the async export completes, images are missing. `MapView.jsx` calls the async export before the map is considered ready, so this is sequenced correctly in normal usage.
- **`mapImages.background` has no icon** — it's used as the cluster icon (shows count text on top).
- **`devicePixelRatio`** (via `prepareIcon` → canvas sizing) is baked into the generated canvases. The `map.addImage(key, value, { pixelRatio: window.devicePixelRatio })` call in `MapView.jsx` line 46 matches.
- **The 4-color scheme maps to device status:** `success` = online, `error` = offline/alarm, `info` = unknown, `neutral` = no-status mode.

## Line index
- 1-29 — SVG imports
- 30-53 — `mapIcons` name→URL map
- 55-65 — `mapIconKey` normalizer
- 67 — `mapImages` mutable dict
- 69-73 — local MUI theme for color resolution
- 75-96 — async default export: image loading + compositing
