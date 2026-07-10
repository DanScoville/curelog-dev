# CLAUDE.md — Tablet build (Galaxy Tab A11 split-layout)

Refines the root `CLAUDE.md` for the tablet truck-app build. This is a truck app (BLE capture + Firebase upload), so **all root hard rules apply** — pressure parsing, no-NOTIFY-on-pressure, temperature stripped from S1 connect, `BTK-` prefix, single-file. The one thing that makes this build different is the **split layout**, and it has bitten us before.

## The split-layout hazard

The tablet lays out two panels side by side, which means page sections are **duplicated**. That duplication creates two failure modes:

1. **Duplicate `id="..."`.** Two copies of a section = two elements with the same ID unless every tablet-side ID is namespaced. Rule: prefix all tablet-section IDs with **`t-`**. An un-prefixed duplicate ID is a bug, not a shortcut.
2. **Single-write updates that only hit one panel.** A plain `getElementById('foo').textContent = ...` updates one copy and leaves the twin stale. Use the **dual-write helpers** — `setText()` and `LOG()` must write to *both* the base element and its `t-` twin.

## ID audit (mandatory before deploy on this build)

Beyond the root ID audit, verify the twins exist: for every namespaced element, confirm both `id="foo"` **and** `id="t-foo"` are present, and that `setText()` / `LOG()` target both. A missing twin renders blank on half the screen with no JS error.

## Note

This directory currently holds only `.gitkeep` — the tablet build hasn't landed here yet. When it does, it starts from the Android app (`apps/android/index.html`) plus the split layout and the `t-` namespacing above.
