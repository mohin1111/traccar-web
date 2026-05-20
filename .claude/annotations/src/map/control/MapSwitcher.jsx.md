# MapSwitcher.jsx

**Role:** Layer-switcher control button that opens a MUI Menu listing available tile styles; selecting one calls `onSelect` to update persisted state in `MapView.jsx`.
**Fits in:** Rendered directly inside `MapView.jsx` JSX (line 145) — the only control rendered as a React child rather than injected via `map.addControl`. Receives `styles`, `selectedId`, and `onSelect` from `MapView`.
**Read next:** [[MapView.jsx]] (renders this; owns `selectedStyleId` state), [[useMapStyles.js]] (provides the `styles` array).

## Public API
- `MapSwitcher({ styles, selectedId, onSelect })` (line 19, default export)
  - `styles` — filtered `StyleDescriptor[]` (only `available` ones from `MapView.jsx` line 73).
  - `selectedId` — currently active style id.
  - `onSelect(id: string)` — called on menu item click.

## Key flows

### Control injection (lines 24-47)
Injects a `maplibregl-ctrl-group` button with `LayersIcon` via `createRoot`. Button `onclick` sets `anchorEl` to the button DOM node (anchor for MUI Menu). Control placed top-right (LTR) / top-left (RTL).

### Menu (lines 49-70)
Standard MUI `Menu` anchored to `anchorEl`. Each style gets a `MenuItem`; selected style shows MUI selected styling. On item click: calls `onSelect(style.id)` then closes menu.

## Gotchas / non-obvious
- **Rendered as a React child of `MapView`**, not via `map.addControl` from outside — it still uses `map.addControl` internally but its JSX (the Menu) lives in the normal React tree, which is why it can use MUI components without portals.
- **`anchorEl` is the raw DOM button** (not a React ref) — MUI Menu positions itself relative to it using `getBoundingClientRect`.
- **`queueMicrotask(() => iconRoot.unmount())`** — same pattern as other controls; defers unmount to avoid React warnings.

## Line index
- 19 — component declaration
- 24-47 — control injection effect (createRoot + MapLibre control)
- 49-70 — MUI Menu JSX with style list
