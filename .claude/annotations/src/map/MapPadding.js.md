# MapPadding.js

**Role:** Applies left (or right for RTL) padding to the map viewport when a side panel (device list) is open, so that map controls and `fitBounds` calculations account for the obscured area.
**Fits in:** Rendered by `MainMap.jsx` whenever the device list panel is visible, with `start` = panel width in pixels.
**Read next:** [[MapView.jsx]] (map singleton), [[MapDefaultCamera.js]] (fitBounds calls benefit from this padding).

## Public API
- `MapPadding({ start })` (line 6, default export) — `start: number` — the pixel offset of the side panel. Returns `null`.

## Key flows

### Padding application (lines 9-21)
`useEffect` on `[start, theme.direction]`:
1. Queries DOM for `.maplibregl-ctrl-top-{start}` and `.maplibregl-ctrl-bottom-{start}` (the MapLibre control corner elements).
2. Sets their `insetInlineStart` CSS to `${start}px` — shifts controls so they don't overlap the panel.
3. Calls `map.setPadding({ left: start })` (or `right` for RTL) — informs MapLibre's camera calculations.
4. Cleanup: resets `insetInlineStart` to 0 and `map.setPadding({ top:0, right:0, bottom:0, left:0 })`.

## Gotchas / non-obvious
- **`insetInlineStart` instead of `left`/`right`** — logical property that respects writing direction automatically; combined with explicit RTL check for `map.setPadding`, this handles both directions.
- **Side effect on DOM elements outside the React tree** — queries MapLibre's internal control containers directly. Fragile if MapLibre changes its class names.
- **Cleanup resets all four padding sides** — not just the side that was set. This is safe (all should be zero when panel is closed) but slightly overreaching.

## Line index
- 6 — component declaration
- 9-21 — effect: control shift + map padding
- 10 — `startKey` direction-aware key selection
- 13-14 — insetInlineStart on control containers
- 15 — map.setPadding call
- 17-20 — cleanup (reset)
