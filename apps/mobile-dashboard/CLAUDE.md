# CLAUDE.md — Mobile Dashboard (phone-sized live monitor)

Refines the root `CLAUDE.md`. Same nature as `apps/dashboard/` — read-only live Firebase monitor — but sized for a phone. Current file: `CureLog Mobile Dashboard - v1.7.19.html` (`VER = '1.7.19-m'`, note the `-m` suffix like the Android app).

## Same rules as the desktop dashboard

- **Firebase JS SDK v8.10.1** (namespaced v8 API), **not** REST. Do not migrate to v9 modular.
- **No BLE / no capture** — read-only. None of the sensor/BLE hard rules apply.
- Chart.js `4.4.1` + `chartjs-plugin-annotation` `3.0.1` from cdnjs.
- Reads `sessions/{TRUCK_ID}/{JOB_NUMBER}/{SHOT_ID}/...` — keep in sync with truck-app data shape.

## Phone-specific

- Layout is single-column and touch-first; keep tap targets large and avoid hover-only affordances.
- Keep it feature-parity-*lite* with the desktop dashboard: show the same data, fewer controls. When porting a desktop-dashboard change here, drop anything that needs screen width.
