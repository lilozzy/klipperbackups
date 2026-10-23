# Changelog

Times are the Pi's local time. Every config edit is also in the git history, through klipper-backup's file watcher.

## 2026-10-22: Revival after 6 months idle
- **Access and backups:** restored SSH access. Fixed klipper-backup, which had stopped on 2026-04-11 because of a corrupt local git object and an expired token.
- **System updates:** Debian bullseye-security moved to archive.debian.org. Fixed `sources.list` and ran a full system upgrade.
- **Software updates:** Klipper v0.13.0-816, Moonraker v0.11.0, Mainsail v2.19.0, crowsnest, KAMP, OctoEverywhere 5.4.0, klipper-backup.
- **Mainboard firmware:** SKR Mini E3 v3 flashed from v0.13.0-435 to v0.13.0-816.
- **LIS2DW firmware:** built and flashed, v0.13.0-464 to v0.13.0-816.
- **Snapshot `01_original` → `02_kamp-m600`:** KAMP added (Line_Purge, Smart_Park; adaptive mesh in `START_PRINT`), and the M600 macro fixed.
- **Moonraker:** `trusted_clients` narrowed (the original is in `snapshots/…moonraker.conf`).
- **Snapshot `03_pid-tuned`:** PID retuned.
  - Hotend at 225 °C: 12.870 / 0.943 / 43.920
  - Bed at 55 °C: 61.059 / 1.938 / 480.838
- **Crash during homing:** the nozzle hit the bed because the accelerometer was mounted on the nozzle (see PROJECT_NOTES §7). No damage.
- **Snapshot `04_pre-shaper`:** temporary `[force_move]` added for testing without homing Z.
- **Snapshot `05_shaper-saved`:** input shaper X 2hump_ei @ 114.4 Hz, Y ei @ 47.0 Hz.
- **Current `printer.cfg`:** `max_accel` 8000 → 4000, `[force_move]` removed, `lis2dw.cfg` include commented out again.
- **Nozzle:** reinstalled and hot-tightened. The Z-offset is **pending recalibration**.
- **Bed leveling, safer order:** because the Z-offset is invalid, `SCREWS_TILT_CALCULATE` was run before `BED_SCREWS_ADJUST`.
  - Bed soaked at 55 °C for 5 minutes.
  - 4 rounds: worst corner 0.24 → 0.16 → 0.065 → 0.02 mm.
- **`PROBE_ACCURACY` at the bed center:** range 0.0025 mm, stdev 0.001 mm.
- **Checkpoint:** stopped before `PROBE_CALIBRATE`.
