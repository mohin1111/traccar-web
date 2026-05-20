# `src/map/control/` — Map control bar components

Five components that inject UI into the MapLibre control bar (top-right/left or bottom corners). All use the same pattern: create a DOM element via `map.addControl(customControl, position)` and mount React content via `createRoot`.

## File index

| File | One-liner | Annotation |
|---|---|---|
| `MapGeocoder.jsx` | Address search via Nominatim; opens Popover with results list | [MapGeocoder.jsx.md](MapGeocoder.jsx.md) |
| `MapNotification.jsx` | Bell icon button; `enabled` prop toggles red highlight | [MapNotification.jsx.md](MapNotification.jsx.md) |
| `MapRuler.jsx` | Click-to-measure ruler; 3 layers (line/point/label); snap-to-position | [MapRuler.jsx.md](MapRuler.jsx.md) |
| `MapSpeedLegend.jsx` | Turbo colormap legend bar (bottom-left) for speed-colored routes | [MapSpeedLegend.jsx.md](MapSpeedLegend.jsx.md) |
| `MapSwitcher.jsx` | Base-tile layer picker; rendered as React child inside `MapView` | [MapSwitcher.jsx.md](MapSwitcher.jsx.md) |

## Shared pattern

All controls (except `MapSpeedLegend`):
```js
useEffect(() => {
  let root;
  const control = {
    onAdd: () => {
      const el = document.createElement('div');
      el.className = 'maplibregl-ctrl maplibregl-ctrl-group';
      const btn = document.createElement('button');
      root = createRoot(btn);
      root.render(<SomeMuiIcon fontSize="small" />);
      el.appendChild(btn);
      return el;
    },
    onRemove: () => {
      queueMicrotask(() => root.unmount());
      el.remove();
    },
  };
  map.addControl(control, position);
  return () => map.removeControl(control);
}, [theme.direction, classes.button]);
```

`MapSpeedLegend` and `MapRuler` follow the same pattern but add MapLibre sources/layers instead of React-rendered icons.

## Conventions

- **`queueMicrotask(() => root.unmount())`** — deferred to avoid React concurrent-mode "unmount during update" warnings.
- **Position** is always `theme.direction === 'rtl' ? 'top-left' : 'top-right'` (or `bottom-*` for `MapSpeedLegend`).
- **`createRoot` in `onAdd`** — allows MUI components (with theme/i18n context) to render inside the MapLibre DOM element. The control is added inside a React effect so the React context providers are already on the tree.
- **Stable-callback refs** (`onClickRef`, `positionsRef`) used in `MapRuler` and `MapNotification` to avoid re-creating MapLibre controls on prop changes.

## Who renders these

| Component | Rendered by |
|---|---|
| `MapGeocoder` | `MainMap.jsx` |
| `MapNotification` | `MainMap.jsx` |
| `MapRuler` | `ReplayPage.jsx` (passes `positions`) |
| `MapSpeedLegend` | `MapRoutePoints.jsx` (conditional) |
| `MapSwitcher` | `MapView.jsx` JSX directly |
