# CLAUDE.md - Tablet build (Galaxy Tab A11+)

Refines the root `CLAUDE.md` for the tablet truck-app build. This is a truck app (BLE capture + Firebase upload), so **all root hard rules apply** - pressure parsing, no-NOTIFY-on-pressure, temperature stripped from S1 connect, `BTK-` prefix, single-file.

## There is no separate tablet file

The tablet build is served from the same file as the Android phone build: **`apps/android/index.html`**. The tablet layout is delivered via a responsive CSS media query, not a copy of the app.

- Breakpoint: `@media (min-width: 900px) and (orientation: landscape)`
- Below the breakpoint (phones, and tablets held in portrait): unchanged single-page tab UX.
- Above the breakpoint (tablets in landscape): Setup panel on the left, Monitor panel on the right, both always visible; Logs and Settings still tab-swap in as full-width overlays.

This directory holds only this doc. Do not add a second `index.html` here - it would fork the code and re-open the divergence problem the responsive approach was chosen to avoid.

## Why the earlier split-layout / `t-` twin plan was dropped

An earlier draft of this doc proposed a duplicated two-panel HTML layout with `t-`-prefixed twin IDs and `setText()` / `LOG()` dual-write helpers. That plan is retired. The responsive single-file approach was chosen because:

- No duplicate IDs and no dual-write helpers means the whole class of "twin went stale" and "duplicate id" bugs cannot happen.
- Every phone-side fix reaches the tablet automatically - no port step.
- Matches the root "single-file architecture stays" rule without additional maintenance overhead.

If you find code referencing `t-`-prefixed IDs or a `setText`/`LOG` dual-write helper, it is a leftover from the retired plan. Remove it or bring it up before adding more.

## What to watch when editing the responsive layout

The split view relies on a few specific CSS behaviors. When touching layout CSS in `apps/android/index.html`, verify these still hold on the Tab A11+ in landscape:

1. `#pages` becomes a two-column grid in split mode. The `.page` elements lose their `position:absolute; inset:0` and become grid items. Any new `.page` sibling must fit this model.
2. `#page-setup` and `#page-monitor` are forced `display:block !important` in split mode. Do not gate their content on `.active` in JS - other pages (Logs, Settings) still use `.active`, but Setup and Monitor are always shown side-by-side.
3. The Logs / Settings overlay uses `:has(#page-logs.active)` / `:has(#page-settings.active)` on `#pages` to hide the split and take over the full grid. If a new page ID is added and should also overlay, extend those `:has()` selectors.
4. `#start-footer` is `position:sticky` at the bottom of the Setup panel in split mode (not `position:fixed` to the viewport). Keep it as a direct child of `#page-setup` so sticky positioning works.
5. `#nav-monitor` is hidden in split mode because both panels are always visible. `#nav-setup` stays visible and acts as the "close overlay / back to split view" affordance when Logs or Settings is open.

## Testing the tablet layout

- On a desktop browser, DevTools device emulation set to ~1200x800 landscape is a reasonable stand-in. Toggling to portrait at the same dimensions should revert to the phone layout.
- On device, the Tab A11+ in Chrome landscape hits the split; rotating to portrait should show the standard tabbed layout.
- BLE behavior is identical to the phone build - same TDWLB5 polling / WETS02 NOTIFY / shared poll loop / 500ms chart timer. If BLE regresses on tablet but works on phone, suspect Chrome-on-tablet quirks, not the layout.
