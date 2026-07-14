# CureLog Backlog

Bug and improvement backlog for the CureLog app suite. Newest audits at the top.
Severity: **HIGH** = wrong data / data loss / crash in normal field use · **MED** = edge-case failure or degraded UX · **LOW** = cosmetic / dead code.
Status: `Open` · `In progress` · `Fixed (<ver>)` · `Won't fix` · `Needs decision`.

---

## Audit 2026-07-14 — full-suite code review

Source: parallel code review of all five app files. HIGH items were re-verified in source by reading the cited lines; items marked *(reviewer)* were surfaced by review but not individually re-opened.

### HIGH

| ID | App | Summary | File:line | Status |
|----|-----|---------|-----------|--------|
| H1 | PC | Corrupted duplicated `<html>`/`#panel-monitor` block; opening Settings or the Monitor tab blanks the live dashboard until reload. `switchTab` re-adds `on` to the outer empty panel. | `apps/pc/PC.html:745-767` | Open |
| H2 | Android | Failed Firebase writes are dropped permanently — `.catch` never re-queues, row already marked `_qu=true`. No offline buffer; every cell/Wi-Fi hiccup = lost cloud data. | `apps/android/index.html:2117-2152` | **Fixed (2.1.62-m)** — per-sensor `s1Up/s2Up` ack flags + in-flight guards; failed writes re-queue and retry every 2s, no dup writes. Field-verify retry after a real dropout. iPad/PC still Open. |
| H3 | Android | 20s data watchdog can never fire for pressure: `runPoll` never sets `A.s1.lastReadMs`, so the `lastReadMs>0` guard is always false in poll mode. Wedged pressure sensor stalls silently. | `apps/android/index.html:1935-1985` vs `2210` | Open |
| H4 | Android | Pressure connect flow *prefers* NOTIFY + silent BE/LE endianness auto-guess — contradicts hard rules #1 (BE only) and #3 (polling only). Live rule violation if TDWLB5 advertises notify; dead code otherwise. | `apps/android/index.html:1500-1517`, `1600-1617` | Needs decision |
| H5 | PC | "Zero" calibration double-counts existing offset (`calOff = -psi` where psi already includes calOff). Wrong PSI after any restored non-zero offset. | `apps/pc/PC.html:1277` | Open |
| H6 | Mobile-dash | Chart never `.destroy()`d; loading a 2nd shot throws "canvas already in use", swallowed in setTimeout, chart stays blank until reload. *(reviewer)* | `.../CureLog Mobile Dashboard - v1.7.19.html:846,949` | Open |

**Cross-app note:** H2, H3, H4 patterns also exist in `apps/ipad/iPad.html` and `apps/pc/PC.html` (forks). Fix android first (live on truck), then port to iPad/PC.

### MEDIUM

| ID | App | Summary | File:line | Status |
|----|-----|---------|-----------|--------|
| M1 | Both dashboards | Temp Min/Max stored in display units, not recomputed on °F↔°C switch → tile shows old number w/ new label AND max stops tracking rest of session. Pressure is immune (stored raw). *(reviewer, found by both)* | dash `~1479`, mobile `~823-827` | Open |
| M2 | All truck apps | Firebase path built from unescaped free-text job/shot/seg/truck; `. # $ [ ] /` break/misfile the REST write (then dropped per H2). PC SDK throws synchronously and kills the drain. | `apps/android/index.html:2565` (+ ipad/pc) | Open |
| M3 | Android | Per-second `_qu` latch set once and never reset; if a drain fires between S1 and S2 writes in a bucket, the later sensor's value for that second is never uploaded. Narrow window. | `apps/android/index.html:2033` | **Fixed (2.1.62-m)** — resolved by the H2 per-sensor tracking (a late second-sensor value now re-queues). iPad/PC still Open. |
| M4 | Dashboard | Chart has no point cap (table capped at 500, chart not); multi-hour session at "all" zoom re-renders 10k+ points every 2s → jank/freeze. *(reviewer)* | `.../curelog-phase1-dashboard-v1.7.21.html:~1532` | Open |

### LOW / dead code *(reviewer-sourced, not individually re-verified)*

| ID | App | Summary | Status |
|----|-----|---------|--------|
| L1 | PC | "About" button calls `openAbout()` but `#about-overlay` markup doesn't exist → dead button. | Open |
| L2 | Android/iPad | Duplicate `toggleConnLog()`; winning def writes to `#su-log-expand-btn` which isn't in the HTML → header tap throws/does nothing. | Open |
| L3 | iPad | Pressure unit "bar" doesn't convert — `psiConv` checks `'Bar'` but dropdown emits lowercase `'bar'`. | Open |
| L4 | Both dashboards | LCL/UCL value of `0` reverts to default via `parseFloat(x) || default` — 0 is a legit limit. | Open |
| L5 | Mobile-dash | Chart UCL/LCL lines ignore pressure-unit conversion (wrong only for Bar/kPa). | Open |
| L6 | Android | Cosmetic doubled label strings ("Reconnect Reconnect", doubled boot banner). | Open |
| L7 | Android | Session-stop "flush upload queue" (Step 4) is guarded by `if (!A.dev && A.fbDb ...)`, but this REST-only app never sets `A.fbDb`, so the explicit end-of-session flush never runs. Periodic 2s drain still covers it until Step 5 stops the timer, but a just-queued batch at stop can be lost. Found while fixing H2. | `apps/android/index.html:2881` | Open |

### Documentation

| ID | Summary | Status |
|----|---------|--------|
| D1 | `CLAUDE.md` Firebase section documents path `sessions/{TRUCK_ID}/{JOB_NUMBER}/{SHOT_ID}/`, but all code uses `shots/{job}/{shot}/{seg}/{truck}/sensor1\|sensor2/readings` (fields `pressRaw`/`tempC`/`tsMs`). Doc is stale — correct it. | Open |

### Suggested fix order
1. Android trio **H2 + H3 + H4** as one commit (force polling, set `lastReadMs`, add write-retry) — confirm H4 direction first.
2. **H1** PC (delete duplicated block).
3. **H5** PC (zero calibration).
4. **M1** dashboards (wrong temp on monitor).
5. **H6 / M4** dashboard chart robustness.
