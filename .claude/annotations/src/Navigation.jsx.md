# Navigation.jsx

**Role:** Full route tree definition for the SPA. Declares all public and protected routes, handles query-param bootstrapping (`?token=`, `?locale=`, `?uniqueId=`, `?openid=`), and lazy-loads every page except `MainPage`.
**Fits in:** Rendered directly by `src/index.jsx` inside `<BrowserRouter>`. Top of the component hierarchy below providers.
**Read next:** [[App.jsx]] (the protected route wrapper rendered at `/`), [[session.js]] (token exchange touches session), [[devices.js]] (`?uniqueId=` dispatches `selectId`)

## Public API
`Navigation` is a default-exported React component with no props. Exclusively used in `src/index.jsx`.

## Route tree summary

### Public routes (no auth required)
- `/login` → `LoginPage` (lazy)
- `/register` → `RegisterPage` (lazy)
- `/reset-password` → `ResetPasswordPage` (lazy)
- `/change-server` → `ChangeServerPage` (lazy)

### Protected routes (nested under `<App>` at `/`)
- `/` index → `MainPage` (**eager** — the only non-lazy page; see Gotchas)
- `/position/:id`, `/network/:positionId`, `/event/:id` — position/event detail viewers
- `/replay` — route replay player
- `/geofences` — geofence manager
- `/emulator`, `/stream` — device emulator, HLS stream
- `/settings/*` — 20+ CRUD pages (devices, users, groups, geofences, notifications, commands, drivers, calendars, computed attributes, maintenances, shares, accumulators, connections)
- `/reports/*` — 12 report pages (combined, chart, events, geofences, route, stops, summary, trips, scheduled, statistics, audit, logs)

All lazy pages share a single `<Suspense fallback={<Loader />}>` at the root (line 123).

## Key flows

### Query-param bootstrapping (`useAsyncTask`, lines 76-117)
Runs only when `hasQueryParams` is true (any of `locale`, `token`, `uniqueId`, `openid` present):
1. `?locale=<code>` — calls `setLocalLanguage(code)`; removes param.
2. `?token=<tok>` — `GET /api/session?token=...` (exchange token for session cookie); removes param.
3. `?uniqueId=<uid>` — `GET /api/devices?uniqueId=...`; dispatches `devicesActions.selectId(items[0].id)` if found; removes param.
4. `?openid=success` — calls `generateLoginToken()` (native bridge); removes param.
5. All params cleaned up with `setSearchParams(newParams, { replace: true })` — no history entry added.

### Loader during param processing (lines 119-121)
While `hasQueryParams` is true (i.e., before the async task finishes), renders `<Loader />` instead of routes. This prevents a flash of the protected page before token exchange completes.

### `<App>` as route element (line 129)
`<Route path="/" element={<App />}>` — `App` is the layout shell for all protected routes. `<App>` renders `<Outlet />` which fills in the matched child route.

## Gotchas / non-obvious
- **`MainPage` is eager-imported** (line 4: `import MainPage from './main/MainPage'`) — the only non-lazy route. This ensures the live map dashboard loads without a Suspense fallback on first authenticated navigation.
- **65 lazy imports** at the top of the file (lines 13-64). Each becomes a separate Vite code-split chunk. On first visit to any lazy page there may be a brief `<Loader />` flash.
- **Token exchange (`?token=`) does NOT dispatch `updateUser`** — it only sets the session cookie via a side-effecting GET. `App`'s own `useAsyncTask` (via `/api/session`) subsequently fetches and stores the user object.
- **`?openid=success`** triggers `generateLoginToken()` — a native bridge call for the Flutter manager wrapper app. In a plain browser context this is a no-op.
- **All protected routes share auth from `<App>`** — if `App` redirects to `/login`, no child page renders. There is no per-route auth check.

## Line index
- 1-11 — imports (react-router, redux, helpers)
- 13-64 — 52 lazy page imports
- 66 — `Navigation` component start
- 72-74 — `hasQueryParams` detection
- 76-117 — `useAsyncTask` for query-param processing
- 84-87 — locale handling
- 89-93 — token exchange
- 95-104 — uniqueId → device select
- 107-112 — openid handler
- 119-121 — loader gate while processing params
- 123-198 — JSX route tree
- 125-128 — public routes
- 129 — `<App>` protected shell
- 130 — `MainPage` eager index route
- 140-180 — settings routes (20+)
- 182-195 — reports routes (12)
