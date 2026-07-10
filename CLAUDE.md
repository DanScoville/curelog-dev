# CLAUDE.md

Guidance for Claude Code working in the **CureLog** repo (`DanScoville/curelog-dev`).

## What this is

CureLog is a suite of **single-file HTML5** field data-acquisition apps for monitoring **CIPP (Cured In Place Pipe)** sewer rehabilitation. Each app connects to BLE sensors on the job truck, captures live **pressure and temperature** during a cure ("shot"), streams readings to **Firebase Realtime Database**, and exports CSV logs. Remote dashboards read the same Firebase data live.

- **Owner:** Dan Scoville, Azuria Water Solutions
- **Hosting:** static files on GitHub Pages — `https://danscoville.github.io/curelog-mobile/`
- **Stack:** vanilla HTML/CSS/JS. **No build system, no bundler, no npm, no frameworks.** Every app is one self-contained `.html` file.

## Repo layout

```
apps/
  android/index.html                    Android Chrome mobile truck app (PWA)   VER 2.1.58-m
  ipad/iPad.html                        iPad truck app (Bluefy browser for BLE) VER 2.1.64-ipad
  pc/PC.html                            Windows/Mac desktop truck app           VER 2.1.19
  dashboard/*.html                      Office/remote live Firebase monitor     VER 1.7.21
  mobile-dashboard/*.html               Phone-sized live dashboard              VER 1.7.19-m
  tablet/                               Galaxy Tab A11 split-layout build
archive/                                Old versioned files
reports/                                Shot reports (roadmap)
scripts/                                Python/bash patching scripts
docs/                                   Reference docs (incl. bom/)
```

Each app file is ~1,300–3,600 lines. The truck app served as `apps/android/index.html` is the primary GitHub Pages entry point; other apps are reached by direct filename URL.

## Hard rules — do not change without explicit instruction from Dan

1. **Pressure parsing (S1 / TDWLB5):** `sint16 big-endian × pressScale`, where `pressScale` comes from the `0x2904` descriptor, **GATT read mode only**. The datasheet "watchlist" parsing applies to *advertisement packets only* and must NOT be used here. This is confirmed correct — do not touch it.
2. **Sensor hardware is fixed:** S1 pressure = **TDWLB5 Gen5** (Transducers Direct, 0–50 PSI, BLE polling). S2 temperature = **WETS02** (Phoenix Sensors, BLE NOTIFY). Do not swap models or parsing without instruction.
3. **No NOTIFY on pressure.** Chrome mis-routes notifications when two devices share the same characteristic UUID. Pressure is **polling only** (`readValue()`, ~250ms). Use IIFE closures and `e.target` matching where same-UUID characteristics are involved.
4. **Temperature is stripped from the S1 connect flow** to avoid GATT contention. Keep it out.
5. **Truck ID prefix is `BTK-`** (Boiler Truck). Not `TVK` — that was fully migrated.
6. **Single-file architecture stays.** No introducing a build step, bundler, or npm dependency.

## Sensor / BLE reference

**S1 — TDWLB5 (pressure):**
- Service UUID `cc4a6a80-51e0-11e3-b451-0002a5d5c51b`
- Pressure characteristic `835ab4c0-51e4-11e3-a5bd-0002a5d5c51b`
- Parse: `sint16 BE × pressScale` (pressScale from `0x2904` descriptor)

**S2 — WETS02 (temperature):**
- Base UUID pattern `A764xxxx-B986-4693-BC60-3028002646F2`
- NOTIFY-only; Cal Period set to 500ms on connect (2 readings/sec)
- Battery % via standard Battery Service

**BLE loop architecture:** single shared `runSharedPollLoop` firing both sensor reads simultaneously via `Promise.all`. Chart history runs on a **separate 500ms timer**, not inside the poll loop. 20-second per-sensor data watchdog; visibility-change reconnect with backoff reset.

**Platform reality:** BLE is reliable on **Chrome/Android**. iOS Safari has no Web Bluetooth — the iPad app requires the **Bluefy** browser and is still limited. React Native is the long-term path to true background BLE on iOS.

## Firebase

- Access is via **REST API only** (`fetch()` with `?auth=` token or open rules) — **no Firebase SDK**.
- RTDB host in use: `curelog-phase1-default-rtdb.firebaseio.com` (project may host multiple DB instances; each app suite can point at its own).
- Uploads are batched in `_uploadQueue`, drained every 2 seconds.
- Data path:
  ```
  sessions/{TRUCK_ID}/{JOB_NUMBER}/{SHOT_ID}/
    _session   { startMs, truck, job, ... }
    readings/  { ts: { psi, temp, ... } }
  ```

## App conventions

- **Version** lives in a `const VER = '...'` at the top of each file **and** (historically) in the filename. Always read `VER` for the current version — it changes most sessions.
- **Changelog** is a `const CHANGELOG = [ ['ver','note'], ... ]` array near the bottom, newest entry **prepended**. The About modal renders from it.
- **ASCII only inside CHANGELOG strings.** Non-ASCII (em-dash, ellipsis, info glyph) has broken the Android Chrome JS parser and silently truncated the changelog. Use `-`, `...`, plain text.
- **Settings** (truck ID, job number, sensor name filters, UCL/LCL) persist in `localStorage`. Never run in Incognito/Private — localStorage is disabled there.
- **Tablet split-layout** duplicates page sections, so IDs are namespaced with a `t-` prefix and `setText()` / `LOG()` dual-write to both copies. Watch for duplicate-ID regressions.

## Editing workflow (proven, reliable)

1. Read the file; the JS is in a single `<script>` block — split on the **last** `<script>` (`content.rfind('<script>')`) into `html_part` + `js`.
2. Make **targeted string replacements**. `assert` the target substring exists before replacing so silent drift fails loud.
3. HTML/CSS edits go in `html_part`; JS edits go in `js`. Recombine as `html_part + '<script>\n' + js + '\n\n</body>\n</html>'`.
4. **Syntax-check JS with `node --check`** before writing out.
5. Bump `VER` and prepend a CHANGELOG entry (ASCII only). Update the filename version if the app uses versioned filenames.
6. **ID audit before deploy:** compare `id="..."` occurrences in HTML against DOM lookups in JS (`getElementById` / `$('id')`) to catch missing elements. On the tablet build, also verify the `t-`-prefixed twins exist.

Prefer small Python/bash patch scripts in `scripts/` over hand-editing thousands of lines.

## Deploy

Upload/commit the new file → push to `main` → GitHub Pages auto-deploys in ~1 minute. No CI build.

## Roadmap / on the horizon

- **Shot report agent (highest ROI):** auto-generate a formatted PDF/HTML report with AI narrative from a completed Firebase session — diagnose pressure violations, temperature excursions, likely cause (crew vs. equipment), annotated charts. Chosen delivery: single-file HTML on GitHub Pages matching the existing stack. Open questions: pressure/temp threshold definitions; synthetic vs. real Firebase data for the first sample.
- Photo upload tied to job sessions (Firebase Storage).
- Daily Firebase shot-report automation (GitHub Actions vs. Cloud Functions vs. Firebase Extensions — undecided).
- Hardware BOM agent + sensor-compatibility screener. An Emerson Rosemount WirelessHART architecture (3051S / 648 / 1410S gateway) was evaluated as an alternative to the TDWLB5/WETS02 BLE path; status vs. the BLE approach is unresolved.

## Working style

Keep changes surgical and single-file. Confirm before altering sensor parsing, BLE routing, or the single-file architecture. When in doubt about a fixed engineering rule above, ask rather than assume.
