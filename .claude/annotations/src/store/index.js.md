# index.js

**Role:** Assembles all Redux slices into one store; attaches `throttleMiddleware`; re-exports every slice's actions as the single import target for the rest of the app.
**Fits in:** Imported as `import store from './store'` by `src/index.jsx` (`<Provider store={store}>`). All action imports in the app use `import { fooActions } from './store'`.
**Read next:** [[session.js]] (richest slice), [[throttleMiddleware.js]] (the custom middleware), [[PACKAGE.md]] (data contract overview)

## Public API
- `default export` — configured Redux store (lines 39-42)
- `errorsActions`, `sessionActions`, `devicesActions`, `eventsActions`, `motionActions`, `geofencesActions`, `groupsActions`, `driversActions`, `maintenancesActions`, `calendarsActions` — all re-exported (lines 28-37)

## Key flows

### Store creation (lines 39-42)
`configureStore` with `combineReducers` of all 10 slices plus `throttleMiddleware` appended after `getDefaultMiddleware()`. RTK Immer + Redux DevTools are active in development (default middleware includes both).

### Single-import action pattern
Every file that dispatches does `import { fooActions } from './store'` (not from `./store/foo`). Exception: `SocketController.jsx` also imports `eventsActions` directly from `./store/events` — cosmetic inconsistency, functionally identical.

## Gotchas / non-obvious
- **`throttleMiddleware` appended, not prepended** — it runs after the default middleware (Immer serialization check, thunk). This means the middleware intercepts the final serialized action, not the Immer draft.
- **No Redux Persist.** State is fully in-memory; a hard reload loses all position history and cached entities. The app re-fetches on mount.

## Line index
- 1-13 — imports of all reducers + throttleMiddleware
- 15-26 — `combineReducers` call
- 28-37 — action re-exports
- 39-42 — `configureStore` with middleware chain
