# Traccar Web — Module Guide

**Repo:** fork of `traccar/traccar-web` at `mohin1111/traccar-web`. Current v6.13.3.
**Stack:** React 19 + Vite 8 + MUI v9 + MapLibre GL 5 + Redux Toolkit 2 + react-router-dom 7. JS (not TS). Apache-2.0.
**Role in Transport OS:** **don't fork-and-reskin** — vendor `src/map/` and `src/resources/l10n/` into our greenfield frontend. Read [`../CLAUDE.md`](../CLAUDE.md) for the wider strategy.

---

## 1. Top-level layout

```
.env                   VITE_* env overrides (not secret)
.github/               CI + translation workflows (build, lint, translation)
.gitignore
.npmrc                 likely engine-strict
.prettierrc.json       singleQuote, printWidth 100
.tx/                   Transifex config for translation pull
.vscode/
eslint.config.js       flat ESLint
index.html             SPA shell with ${title}/${colorPrimary}/${description} placeholders
package.json           npm; no engines field
package-lock.json      lockfile (use npm)
public/                favicon, logo.svg, PWA PNGs, styles.css
README.md
simple/                separate minimal SPA (not the main app)
src/                   all React source
vite.config.js         Vite + PWA + static-copy
```

`.github/workflows/`:
- `build.yml` — `npm ci && npm run build` on push/PR
- `lint.yml` — `npm ci && npm run lint` on push/PR
- `translation.yml` — manual; Transifex CLI pulls translations and commits as Traccar Bot

## 2. Build & run

| Task | Command |
|---|---|
| Install | `npm ci` (lockfile-driven) |
| Dev server | `npm start` (`vite --host`, port 3000) |
| Production build | `npm run build` |
| Lint | `npm run lint` (zero warnings allowed) |
| Lint fix | `npm run lint:fix` |
| PWA icons | `npm run generate-pwa-assets` |

**Vite dev proxy** (`vite.config.js`):
```
'/api/socket' → ws://localhost:8082
'/api'        → http://localhost:8082
```
Backend (server module) must be running at `localhost:8082`.

**Node:** no `engines` field; CI uses `actions/setup-node@v6` (LTS).
**Output:** `build/` (Vite `outDir`).

**PWA:** Workbox via `vite-plugin-pwa`. Precaches `**/*.{js,css,html,woff,woff2,mp3}`. Navigate fallback denies `/api/**`. RTL plugin JS (`mapbox-gl-rtl-text.js`) copied to build root via `vite-plugin-static-copy`. Manifest placeholders (`${title}`/`${description}`/`${colorPrimary}`) filled by Java backend at serve time.

## 3. Routing & top-level layout

Entry: `src/index.jsx` → `<ServerProvider>` → `<LocalizationProvider>` → `<AppThemeProvider>` → `<BrowserRouter>` → `<Navigation>`.

**Public routes (lazy):**
- `/login` → `LoginPage`
- `/register` → `RegisterPage`
- `/reset-password` → `ResetPasswordPage`
- `/change-server` → `ChangeServerPage`

**Protected routes (nested under `<App>`, which fetches `/api/session` and redirects to `/login` on 401):**
- `/` (index) → **`MainPage` — eager loaded**, live map dashboard
- `/position/:id` → `PositionPage` (full attribute viewer)
- `/network/:positionId` → `NetworkPage` (cell/wifi debug)
- `/event/:id` → `EventPage`
- `/replay` → `ReplayPage` (route replay player)
- `/geofences` → `GeofencesPage`
- `/emulator` → `EmulatorPage`
- `/stream` → `StreamPage` (HLS video)
- `/settings/*` — 20+ CRUD pages (devices, users, groups, geofences, notifications, commands, drivers, calendars, computed attrs, maintenances, shares, accumulators, connections)
- `/reports/*` — 12 report pages (combined, chart, events, geofences, route, stops, summary, trips, scheduled, statistics, audit, logs)

All routes except `MainPage` are `React.lazy()` + `<Suspense fallback={<Loader />}>`.

`Navigation.jsx` also handles query-param bootstrapping: `?token=` (auth exchange), `?locale=`, `?uniqueId=` (device select), `?openid=success`.

## 4. State management

**Store:** `src/store/index.js` — `configureStore` + `combineReducers` + custom `throttleMiddleware`.

| Slice | Purpose |
|---|---|
| `session.js` | `server`, `user`, `socket` status, live `positions` map (deviceId→position), `history` (live route), `logs` |
| `devices.js` | `items` (id→device), `selectedId` |
| `events.js` | unread event queue for the drawer |
| `geofences.js` | id→geofence |
| `groups.js` | id→group |
| `drivers.js` | id→driver |
| `maintenances.js` | id→maintenance |
| `calendars.js` | id→calendar |
| `motion.js` | motion state for MotionController |
| `errors.js` | global error queue |
| `throttleMiddleware.js` | rate-limits rapid position dispatches |

**Session bootstrap:**
1. `ServerProvider` → `GET /api/server` → `sessionActions.updateServer()`
2. `App` → `GET /api/session` → `sessionActions.updateUser()` (or redirect to login)
3. `CachingController` → fetches geofences/groups/drivers/maintenances/calendars
4. `SocketController` → `GET /api/devices` then opens WebSocket

## 5. API client

**No central API client class.** All REST via native `fetch` through:
- `src/common/util/fetchOrThrow.js` — thin wrapper that throws on non-OK
- Raw `fetch` for non-throwing cases (session check, socket reconnect)

**Auth:** Session-cookie based. No manual token headers; browser sends cookies automatically.

**WebSocket** (`src/SocketController.jsx`):
- Connects to `ws[s]://host/api/socket` when authenticated
- Messages: `{devices}` → `devicesActions.update`, `{positions}` → `sessionActions.updatePositions`, `{events}` → event queue + optional alarm audio, `{logs}` → `sessionActions.updateLogs`
- Auto-reconnects on close (60s delay) with REST poll fallback (`/api/devices` + `/api/positions`)
- Reconnects on `window.online` and `visibilitychange`
- `{logs: true/false}` toggles server-side log streaming

## 6. Map subsystem — the most reusable part

Root: `src/map/`.

### `core/`
| File | Purpose |
|---|---|
| `MapView.jsx` | **Main MapLibre wrapper.** Single global `maplibregl.Map` attached to a detached DOM element; mounts/unmounts into whichever React container is active. Google protocol registration, style switching, RTL plugin, attribution/navigation controls. |
| `useMapStyles.js` | **Tile/style provider registry.** 26 providers. |
| `mapUtil.js` | `geofenceToFeature`, `geometryToArea`, `findFonts`, coordinate helpers |
| `preloadImages.js` | Preloads SVG marker icons as MapLibre sprites; exports `mapIconKey`, `mapImages` |

**Style providers (26):** OpenFreeMap · LocationIQ Streets · LocationIQ Dark · OSM · OpenTopoMap · Carto · Google Road/Satellite/Hybrid · MapTiler Basic/Hybrid · Bing Road/Aerial/Hybrid · TomTom · HERE Basic/Hybrid/Satellite · Yandex · AutoNavi · Ordnance Survey · Mapbox Streets/Dark/Outdoors/Satellite · **Custom URL** (MapmyIndia/Ola slot in here).

### `control/`
`MapSwitcher.jsx` (layer switcher button), `MapGeocoder.jsx` (LocationIQ search), `MapRuler.jsx`, `MapNotification.jsx` (toast anchored to map), `MapSpeedLegend.jsx`.

### `draw/`
`MapGeofenceEdit.js` — `@mapbox/mapbox-gl-draw` (patched for MapLibre CSS classes); polygon/line/trash controls; on `draw.create` POSTs to `/api/geofences`; on `draw.update`/`draw.delete` PUTs/DELETEs. `theme.js` — custom draw styles.

### `main/`
`MapDefaultCamera.js`, `MapSelectedDevice.js`, `MapLiveRoutes.js` (renders live route polylines from `session.history`), `MapAccuracy.js`, `PoiMap.js`.

### Top-level `/src/map/`
| File | Purpose |
|---|---|
| `MapPositions.js` | **Core marker/cluster layer.** GeoJSON source with `cluster:true`; symbol layers for icons + direction arrows; separate selected-device layer; click callbacks. |
| `MapMarkers.js` | Generic static marker layer (route start/end) |
| `MapGeofence.js` | Fill + line + title layers for geofences from Redux |
| `MapRoutePath.js` | Speed-color-coded replay path |
| `MapRouteCoordinates.js` | Route data management for replay |
| `MapRoutePoints.jsx` | Start/end JSX markers for replay |
| `MapCamera.js` | `fitBounds` helper |
| `MapCurrentLocation.js` | Browser geolocation dot |
| `MapPadding.js` | Side-panel-aware padding |
| `MapScale.js` | Unit-aware scale bar |

### `overlay/`
`useMapOverlays.js` — 12 raster overlays: Google Traffic, OpenSeaMap, OpenRailwayMap, 5× OpenWeather (clouds/precip/pressure/wind/temp), TomTom Flow/Incidents, HERE Flow, Custom URL. `MapOverlay.js` — renders the active one.

**Live update mechanism:** WebSocket `positions` → `sessionActions.updatePositions` → `MapPositions` calls `map.getSource(id).setData()` in-place (no full re-render).

## 7. Screens

### `src/main/` — 8 files
Key: `MainPage.jsx`, `MainMap.jsx`, `DeviceList.jsx`. Live-tracking dashboard: split map+list, device filter (`useFilter.js`), virtualised rows (`DeviceRow.jsx`, `react-window`), events drawer (`EventsDrawer.jsx`), motion (`MotionController.jsx`).

### `src/login/` — 6 files
`LoginPage.jsx`, `LoginLayout.jsx`, `RegisterPage.jsx`. Email+password + token + OpenID, registration, password reset, server URL change.

### `src/settings/` — 32 files
`DevicePage.jsx`, `UserPage.jsx`, `ServerPage.jsx`. Full CRUD for every server entity.

### `src/reports/` — 12 files
`PositionsReportPage.jsx`, `CombinedReportPage.jsx`, `ChartReportPage.jsx`. Trips, stops, events, geofence crossings, positions, summary, chart (Recharts), scheduled, statistics, audit, logs. Excel export via `exceljs`.

### `src/other/` — 7 files
`ReplayPage.jsx`, `GeofencesPage.jsx`, `PositionPage.jsx`, `EventPage.jsx`, `NetworkPage.jsx`, `EmulatorPage.jsx`, `StreamPage.jsx`.

## 8. i18n

- 61 JSON files in `src/resources/l10n/`
- `en.json` has 659 strings (sets the canonical key set)
- **Indian languages present:** `hi.json` (Hindi), `ta.json` (Tamil), `bn.json` (Bengali), `ml.json` (Malayalam), `ne.json` (Nepali — note: Nepal, not India)
- **Missing for India:** Telugu, Marathi, Gujarati, Kannada, Punjabi, Urdu

**Lazy-loading** (`src/common/components/LocalizationProvider.jsx`):
- English bundled eagerly
- Others via `import.meta.glob(...)` → Vite per-locale chunks
- React 19 `use()` hook suspends until promise resolves
- Priority: user attr `language` > server attr `language` > localStorage > `navigator.languages`

## 9. Theming & branding

`src/common/theme/index.js` builds `createTheme({palette, direction, dimensions, components})`. Re-memoizes on `server`/`darkMode`/`direction` change.

`palette.js`:
- `primary.main` ← `server.attributes.colorPrimary` (validated hex) or MUI indigo
- `secondary.main` ← `server.attributes.colorSecondary` or MUI green
- `mode` ← `server.attributes.darkMode` or `prefers-color-scheme`

**`index.html` placeholders** filled by Java backend: `${title}`, `${description}`, `${colorPrimary}`.

**Logo:** `public/logo.svg` (PWA icon source); `src/login/LogoImage.jsx` falls back to `/logo.svg` if no server URL.

**RTL:** stylis-plugin-rtl via Emotion; map controls mirror positions.

## 10. PWA / offline

- Workbox via `vite-plugin-pwa` — precaches all JS/CSS/HTML/woff/mp3
- Navigate fallback denies `/api/**`
- Manifest icons: 64/192/512px (maskable) from `public/`
- `UpdateController.jsx` watches new SW registration and prompts reload

## 11. Tests / quality

- **Zero test files.** No Vitest/Jest/Playwright. No CI test job.
- **ESLint flat config** (`eslint.config.js`): `@eslint/js`, `@eslint-react/eslint-plugin`, `eslint-plugin-import-x`, `eslint-plugin-react-hooks`, `eslint-plugin-prettier`. `--max-warnings 0` in CI.
- **Prettier:** `singleQuote: true`, `printWidth: 100`.

## 12. Vendor-able subsections

### Clean to vendor (low coupling)

**`src/resources/l10n/`** — pure JSON. Drop into any project. Replicate loader from `LocalizationProvider.jsx`.

**`src/resources/images/`** — SVG/PNG marker assets. Self-contained.

**`src/common/util/`** — most are pure functions (`formatter.js`, `colors.js`, `converter.js`, `duration.js`, `stringUtils.js`). `preferences.js` and `usePersistedState.js` depend on Redux/localStorage; `fetchOrThrow.js` is 5 lines.

### `src/map/` — liftable as a unit (with companions)

Cannot be a pure drop-in because of imports outside `src/map/`. **To vendor `src/map/` whole, also copy:**
- `src/resources/images/` (all SVGs)
- `src/common/util/preferences.js` + `usePersistedState.js`
- `src/common/util/useFeatures.js`, `colors.js`, `formatter.js`
- `src/reactHelper.js` (`useAsyncTask`, `useCatchCallback`)
- `src/common/components/LocalizationProvider.jsx` (or stub `useAttributePreference` + `useTranslation`)
- Redux slices: `session`, `geofences`, `devices`, `errors`

The map module is **well-encapsulated within its own directory** — internal cross-refs stay inside `src/map/`. External coupling is exclusively to MUI (theming), react-redux (data), react-router (draw navigation), and the common util/resource folders.

**`src/map/core/useMapStyles.js` alone** is especially portable — just stub `useAttributePreference` + `useTranslation` and you have a 26-provider tile registry. MapmyIndia/Ola Maps slot in trivially via Custom URL.

## 13. Things future-Claude must know

**Read first, in this order:**
1. `src/Navigation.jsx` — full route tree
2. `src/App.jsx` — auth guard + controller composition
3. `src/SocketController.jsx` — real-time data flow (WebSocket → Redux → map)
4. `src/map/core/MapView.jsx` + `src/map/MapPositions.js` — together form the live tracking core

**Pattern for adding a new map layer:** import the global `map` singleton from `MapView.jsx`; in a `useEffect`, call `map.addSource(id, ...)` + `map.addLayer(id, ...)`; update on state change via `map.getSource(id).setData(...)`; clean up in effect cleanup.

**Branding from server config:** `state.session.server.attributes` carries `colorPrimary`/`colorSecondary`/`logo`/`title`. No code changes needed to rebrand at runtime — only server-side config.

**License notice:** Apache-2.0. Free to fork, modify, redistribute (closed or open), and SaaS. Just preserve NOTICE.
