# App.jsx

**Role:** Protected route wrapper and auth guard. Fetches `/api/session` to validate the cookie; dispatches `updateUser` on success or redirects to `/login` (or `/register` for new servers) on 401. Also renders the four invisible controller components and the responsive bottom menu.
**Fits in:** Rendered as the `element` of the root `<Route path="/">` in `Navigation.jsx`. All protected page routes are children of this component via `<Outlet />`.
**Read next:** [[Navigation.jsx]] (places `<App>` in the route tree), [[SocketController.jsx]] (rendered inside App), [[session.js]] (the slice App reads/writes)

## Public API
`App` is a default-exported React component with no props. It is exclusively used as a route element in `Navigation.jsx`.

## Key flows

### Auth check on mount (`useAsyncTask`, lines 50-67)
Runs when `user` is null (not yet authenticated):
1. `GET /api/session` with AbortSignal.
2. If `200 OK`: `dispatch(sessionActions.updateUser(userJson))` → user state populated → component re-renders past the null guard.
3. If non-OK: saves `window.location.pathname + search` in `sessionStorage` as `postLogin` (for redirect-after-login), then `navigate('/register')` if `server.newServer` is true, else `navigate('/login')`.

### Null guard + loading state (lines 69-71)
While `user == null`, returns `<Loader />`. This prevents children (and the four controllers) from rendering before auth is confirmed.

### Terms-of-service gate (lines 72-74)
If `server.attributes.termsUrl` is set and `user.attributes.termsAccepted` is falsy, renders `<TermsDialog>` instead of the page. Accepting calls `PUT /api/users/:id` with `termsAccepted: true` and updates the user in Redux.

### Controller composition (lines 76-89)
Once authenticated and terms accepted, renders (all invisible/renderless):
- `<SocketController />` — WebSocket lifecycle, live data
- `<CachingController />` — fetches geofences/groups/drivers/maintenances/calendars
- `<UpdateController />` — PWA service-worker update detection
- `<MotionController />` — maintenance-due computation
Then `<Outlet />` (the active child route page) and, on mobile, `<BottomMenu />`.

### `acceptTerms` (lines 41-48)
`PUT /api/users/:id` with the full user object + `termsAccepted: true`. Updates Redux user on success.

## Gotchas / non-obvious
- **`newServer` drives login vs register redirect** (line 61): `state.session.server.newServer` is `true` when the server has no admin account yet — first-run state. Checking this before redirecting avoids confusion on initial setup.
- **`postLogin` sessionStorage key** enables post-auth deep-link restoration. `LoginPage` reads this and redirects after successful login.
- **The four controllers are renderless** — they return `null` (or pure JSX overlay in SocketController's case for Snackbar). They could be hooks but are components to leverage React's `useEffect` cleanup lifecycle tied to auth state.
- **`useAsyncTask` dependency on `user`** means the fetch re-runs if `user` is reset to null (logout from another tab/code path). Signal-based abort prevents race conditions on fast navigation.

## Line index
- 37 — `newServer` selector (controls login vs register redirect)
- 38 — `termsUrl` selector
- 39 — `user` selector
- 41-48 — `acceptTerms` handler (PUT + updateUser)
- 50-67 — auth check `useAsyncTask`
- 53 — `GET /api/session`
- 57-62 — redirect on auth failure (save postLogin, navigate)
- 69-71 — null guard → `<Loader />`
- 72-74 — terms gate → `<TermsDialog />`
- 76-89 — controller + outlet render (authenticated state)
