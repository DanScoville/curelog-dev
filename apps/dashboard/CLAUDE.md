# CLAUDE.md — Dashboard (office / remote live monitor)

Refines the root `CLAUDE.md` for this directory. Where the two differ, this file wins for dashboard work.

## What this app is

Read-only live monitor of Firebase session data for office/remote viewers. It does **not** connect to BLE, does **not** capture or upload readings — it only *reads* what the truck apps wrote. Current file: `curelog-phase1-dashboard-v1.7.21.html` (`VER = '1.7.21'`).

## Key differences from the truck apps

- **Firebase JS SDK, not REST.** Unlike the truck apps (REST-only, no SDK — root hard rule #1 area), the dashboard loads the **Firebase JS SDK v8.10.1** from `gstatic.com` (`firebase-app.js` + `firebase-database.js`) and subscribes to live updates. Keep it on the **v8 namespaced API** (`firebase.database()...`) — do not "upgrade" to the v9 modular SDK; that would be a rewrite, not a patch.
- **No sensor/BLE code.** None of the BLE hard rules (pressure parsing, no-NOTIFY, GATT contention) apply here. Do not add BLE.
- **Charting:** Chart.js `4.4.1` + `chartjs-plugin-annotation` `3.0.1`, both from cdnjs. Annotations draw UCL/LCL and event markers. Zoom controls are custom.

## Conventions still in force

- Single-file HTML, no build step (root rule holds).
- `const VER` at top; `const CHANGELOG` prepend-newest, **ASCII only** (root rule holds).
- Reads the same Firebase path as the truck apps: `sessions/{TRUCK_ID}/{JOB_NUMBER}/{SHOT_ID}/{_session, readings}`. If the truck-app data shape changes, update the dashboard parser to match.
