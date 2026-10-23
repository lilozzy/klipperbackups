# Ender 3 Pro — Klipper config 🖨️

Klipper configuration and calibration history for a heavily modified **Creality Ender 3 Pro**, running on a Raspberry Pi 3B with MainsailOS.
[Klipper-Backup](https://github.com/Staubgeborener/Klipper-Backup) pushes changes here automatically: on every config edit, and every 4 hours.

## Status

| Area | State |
|---|---|
| Software | ✅ Klipper v0.13.0-816 on the host, the SKR mainboard and the accelerometer. Moonraker v0.11, Mainsail v2.19. |
| Backups | ✅ Working again (repaired 2026-10-22) |
| PID (hotend and bed) | ✅ Tuned 2026-10-22 |
| Input shaping | ✅ X 2hump_ei @ 114.4 Hz, Y ei @ 47.0 Hz; `max_accel` 4000 |
| Bed level (screws tilt) | ✅ Trammed 2026-10-22 to 0.02 mm. Probe repeatability 0.0025 mm range. |
| Z-offset | ⚠️ **Needs recalibration.** The nozzle was reinstalled. Do not print until it's redone. |
| Part-fan tuning | ⏳ To do: blower stall point, `off_below` / `kick_start_time` |
| OrcaSlicer calibration | ⏳ To do: temperature, max flow, flow ratio, PA, retraction (PLA, then ASA) |

## Hardware

- **Board:** BTT SKR Mini E3 v3
- **Hotend:** Micro Swiss all-metal hotend, 0.4 hardened steel nozzle, 60 W heater, Bowden extrusion
- **Cooling:** dual 5015 part-cooling blowers
- **Probe:** CR/BLTouch
- **Accelerometer:** BTT LIS2DW (USB, connected only for input shaping)
- **Bed:** textured PEI
- **Materials:** PLA (main), ASA (occasional)

The full hardware list is in [PROJECT_NOTES.md](printer_data/config/PROJECT_NOTES.md#1-hardware).

## Repository layout

```
printer_data/config/
├── printer.cfg              # main Klipper config (SAVE_CONFIG block holds PID, shaper, mesh, probe offset)
├── PROJECT_NOTES.md         # full project document: hardware, versions, tuning, procedures, lessons, to-do
├── history/
│   ├── CHANGELOG.md         # what changed and when
│   ├── snapshots/           # printer.cfg / moonraker.conf at each milestone
│   └── input_shaper/<date>/ # graphs, results summary, raw accelerometer data
├── mainsail.cfg, KAMP_Settings.cfg, lis2dw.cfg, crowsnest.conf, moonraker.conf, ...
```

`*.csv`, `*.bak` and Klipper's own `printer-<timestamp>.cfg` files are excluded by the backup rules. That's why the snapshots and raw data under `history/` use different file names.

## Links

- [Project notes](printer_data/config/PROJECT_NOTES.md)
- [Changelog](printer_data/config/history/CHANGELOG.md)
- [Input shaper results 2026-10-22](printer_data/config/history/input_shaper/2026-10-22/README.md)
