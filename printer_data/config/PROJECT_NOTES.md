# Ender 3 Pro / Klipper — Project Notes

Living record of the hardware, software, calibration results and open work for this printer. History: [CHANGELOG](history/CHANGELOG.md), [config snapshots](history/snapshots/), [input shaper results](history/input_shaper/2026-10-22/README.md).
Last checkpoint: **2026-10-22**. This file lives in `printer_data/config/`, so klipper-backup pushes it with the config.

> ⚠️ **Before the next print:** the nozzle was removed and re-tightened on 2026-10-22. The saved Z-offset is no longer valid. Redo `PROBE_CALIBRATE` (see [Calibration procedure](#calibration-procedure)).

---

## 1. Hardware

| Part | Details |
|---|---|
| Printer | Creality Ender 3 Pro, Bowden extrusion |
| Mainboard | BIGTREETECH SKR Mini E3 v3.0 (STM32G0B1, 8 KiB bootloader, 8 MHz crystal, USB) |
| Host | Raspberry Pi 3B running MainsailOS 1.3.2 (Debian 11 bullseye) |
| PSU | Mean Well LRS-350-24 (24 V, 14.6 A) |
| Hotend | Micro Swiss all-metal hotend, silicone sock fitted |
| Nozzle | 0.4 mm hardened steel |
| Heater cartridge | 24 V **60 W** (stock was 40 W) |
| Hotend thermistor | Generic "NTC 3950 100k" Ender-3 replacement, configured as `EPCOS 100K B57560G104F` (curve difference vs `Generic 3950` is ≤ ~2.7 °C over 55–260 °C, so left as is) |
| Probe | CR/BLTouch, offsets x −50, y −7 |
| Part cooling | Dual 50 mm 5015 blowers (24 V), both on the single part-fan header `PC6` |
| Hotend fan | Upgraded fan on `PB15` (`heater_fan`, on above 50 °C) |
| Electronics fan | `PC7`, `temperature_fan` on Pi CPU temp, 60 °C target |
| Accelerometer | BTT LIS2DW v1.0 ("S2DW"), USB RP2040. **Only connected for input shaping.** |
| Webcam | Logitech C930e (crowsnest / ustreamer) |
| Display | BTT TFT in emulated 12864 mode |
| Bed | Textured PEI sheet |
| Tools | Feeler gauge set 0.04–0.88 mm (use the **0.10 mm** blade for Z-offset) |

Materials: mostly **PLA** (Overture Matte PLA), occasionally **ASA**. No PETG.

---

## 2. Software (as of 2026-10-22 update)

| Component | Version |
|---|---|
| Klipper (host) | v0.13.0-816 |
| Mainboard firmware | v0.13.0-816 |
| Accelerometer firmware | v0.13.0-816 |
| Moonraker | v0.11.0 |
| Mainsail | v2.19.0 |
| crowsnest | v4.2.0 |
| KAMP | v1.1.2 (Line_Purge and Smart_Park only) |
| klipper-backup | v1.3.6 (cron every 4 h plus file watcher) |
| OctoEverywhere | 5.4.0 |
| Slicer | OrcaSlicer 2.3.1 on the desktop PC |

**Note on Debian 11 (bullseye) end of life:** the security packages moved to `archive.debian.org`. `/etc/apt/sources.list` was switched to the archive source so system updates work again.

### Firmware builds
These live on the Pi in `~/firmware/`; see `~/firmware/README.txt` for details.

- **SKR Mini E3 v3**
  - Config: `config.skr-mini-e3-v3`
  - Rollback image: `skr-mini-e3-v3_v0.13.0-435.bin`
  - Flash via the SD card, with Klipper stopped:
    ```
    ./scripts/flash-sdcard.sh -f ~/firmware/out-skr-mini-e3-v3/klipper.bin /dev/serial/by-id/usb-Klipper_stm32g0b1xx_<id>-if00 btt-skr-mini-e3-v3
    ```
- **LIS2DW (RP2040)**
  - Config: `config.btt-lis2dw-rp2040`
  - **If it already runs Klipper**, flash over USB (needs sudo):
    ```
    cd ~/klipper && make flash KCONFIG_CONFIG=~/firmware/config.btt-lis2dw-rp2040 OUT=~/firmware/out-lis2dw/ FLASH_DEVICE=/dev/serial/by-id/usb-Klipper_rp2040_<id>-if00
    ```
  - **From BOOT mode** (hold BOOT while plugging in; it shows up as "RP2 Boot"):
    ```
    sudo ~/klipper/lib/rp2040_flash/rp2040_flash ~/firmware/out-lis2dw/klipper.uf2
    ```
    Use the **`.uf2`** file, not the `.elf`.
- **After any Klipper update,** rebuild both firmwares from `~/klipper`:
  ```
  make KCONFIG_CONFIG=<config> OUT=<outdir>/
  ```
  Then reflash, so the board firmware matches the host version.

---

## 3. Current tuning values

| Setting | Value | Date / notes |
|---|---|---|
| Hotend PID | Kp 12.870, Ki 0.943, Kd 43.920 | 2026-10-22, tuned at 225 °C with the part fan off. Previously 21.609 / 1.533 / 76.170, which predates the 60 W heater. |
| Bed PID | Kp 61.059, Ki 1.938, Kd 480.838 | 2026-10-22, tuned at 55 °C. Previously 54.027 / 0.770 / 948.182. |
| Input shaper X | `2hump_ei` @ 114.4 Hz | 2026-10-22. Resonance peaks at 67 Hz and 108 Hz. Previously mzv @ 78. |
| Input shaper Y | `ei` @ 47.0 Hz | 2026-10-22. Sensor taped to the bed; two runs agreed (47.0 and 49.4 Hz). Previously mzv @ 29.4, which was wrong. |
| `max_accel` | 4000 mm/s² | 2026-10-22, the Y-shaper limit. Previously 8000. |
| `max_velocity` | 400 mm/s | |
| Extruder `rotation_distance` | 29.172 | |
| BLTouch `z_offset` | ~~0.140~~ **must be redone** | The nozzle was reinstalled 2026-10-22. |
| Pressure advance | Set by the Orca filament profile (`SET_PRESSURE_ADVANCE ADVANCE=0.4`) | 0.4 is plausible for Bowden; to be re-tested. |
| Firmware retraction | 1.4 mm @ 20 mm/s | Plan is to move retraction into Orca (see §6). |
| TMC2209 currents | X/Y/Z 0.58 A, E 0.65 A; stealthChop on all | |

### Bed / probe geometry
- `safe_z_home`: 115,115, with a z_hop of 10 mm.
- `bed_mesh`: 5×5 points, from (2,10) to (195,223). The adaptive mesh comes from Klipper's built-in `BED_MESH_CALIBRATE ADAPTIVE=1`.
- `bed_screws` (nozzle positions): 34,31 / 200,31 / 200,198 / 34,198.
- `screws_tilt_adjust` (probe positions): 81,35 / 249,35 / 249,206 / 81,206, `CW-M4` thread.

---

## 4. Config changes made during this session

1. **KAMP added.**
   - `[include KAMP_Settings.cfg]`, using Line_Purge and Smart_Park.
   - The `START_PRINT` order is now:
     1. Home.
     2. Wait for the bed to reach temperature.
     3. `BED_MESH_CLEAR`, then `BED_MESH_CALIBRATE ADAPTIVE=1`.
     4. `SMART_PARK`.
     5. Heat the nozzle with `M109`.
     6. `LINE_PURGE`.
2. **M600 fixed.**
   - Removed references to a filament sensor that doesn't exist.
   - The Z lift is limited to the travel remaining before max Z.
   - Uses relative extrusion.
   - Resume with `SWAP_RESUME` or Mainsail's RESUME.
3. **Moonraker `trusted_clients` narrowed** to the printer/desktop subnet, the management host and localhost.
4. **Input shaper and `max_accel` updated** (see §3). The old X and Y lines in the main `[input_shaper]` section are commented out, and the values now live in the SAVE_CONFIG block.
5. **`lis2dw.cfg` `probe_points`** changed to 117,117,20 (bed center). Its include stays **commented out** whenever the sensor is unplugged, otherwise Klipper won't start.
6. **A temporary `[force_move]` was added and then removed.** It was only needed for resonance tests with the sensor on the nozzle (see §7).

---

## 5. Calibration procedure

### Mechanical / first layer (do these in order)
1. Heat the bed to 55 °C and let it soak for about 5 minutes.
2. Run `BED_SCREWS_ADJUST`, using the 0.10 mm feeler gauge at each screw.
3. Run `SCREWS_TILT_CALCULATE`, adjusting and repeating until every screw reads within about 00:02.
4. Run `PROBE_ACCURACY`. The range should be ≤ 0.01–0.02 mm.
5. `PROBE_CALIBRATE`:
   - Bed at 55 °C, nozzle at about 150 °C, nozzle tip wiped clean.
   - Step down until the 0.10 mm gauge drags slightly, then `ACCEPT`.
   - **Add the gauge thickness to the result.** Example: if Klipper reports 1.20, save 1.30.
   - Then `SAVE_CONFIG`.
6. Fine-tune Z from the first-layer test in steps of 0.01–0.02 mm.

### Input shaping (accelerometer)
1. Plug the sensor into a **Pi** USB port. The SKR's USB port is device-only and won't work. Use a USB-C cable that carries **data**: the first cable tried only carried power.
2. Uncomment `[include lis2dw.cfg]`, restart Klipper, then run `ACCELEROMETER_QUERY` and `MEASURE_AXES_NOISE`.
3. **X axis:** mount the sensor on the toolhead and run `SHAPER_CALIBRATE AXIS=X`.
4. **Y axis:** the bed is the Y axis on an Ender 3, so the sensor goes **on the bed**. Then run `SHAPER_CALIBRATE AXIS=Y`.
5. Run `SAVE_CONFIG`, comment the include out again, and unplug the sensor.
6. Make sure the printer is standing on its own feet with nothing touching the frame during the test.

### PID
- `PID_CALIBRATE HEATER=extruder TARGET=225`, then `PID_CALIBRATE HEATER=heater_bed TARGET=55`, then `SAVE_CONFIG`.
- These only heat, so they're safe to run remotely.

---

## 6. OrcaSlicer notes

Printer profile: **"(Kevins)Creality Ender-3 Pro 0.4 nozzle (accel and speed)"**. Physical printer type: Octo/Klipper, host `klipperpi.lil-oz.com` (HTTP).

**Problems found in the current Orca setup:**

| Problem | Effect | Fix |
|---|---|---|
| The start G-code runs its own `BED_MESH_CALIBRATE ... ADAPTIVE=0` **before the bed is hot**, and doesn't call `START_PRINT`. | The mesh is measured on a cold bed, so it's wrong once the bed heats up. `START_PRINT`/KAMP is never used. | Either wait for the bed temperature before meshing in Orca, or replace the start G-code with `START_PRINT BED=[bed_temperature_initial_layer_single] EXTRUDER=[nozzle_temperature_initial_layer]`. |
| `use_firmware_retraction = 1` | Orca's retraction settings and calibration don't change what gets printed. The March retraction tower (0.1–0.8 mm) never applied. | Turn firmware retraction off and set retraction per filament in Orca. Expect a Bowden range of about 1.5–4 mm, kept modest for the all-metal hotend. |
| `filament_max_volumetric_speed = 200` | No real flow limit. The hardened steel nozzle realistically manages about 8–12 mm³/s. | Measure it with Orca's max-flow test, starting from a placeholder of about 10. |
| Pressure advance is set in the filament profile (0.4) | Fine, but it needs to be retested after the shaper and acceleration changes. | Orca's PA test. |
| `max_accel` was 8000 | Ringing on Y. | Now 4000 in Klipper. Orca's acceleration values of 0 follow Klipper's limit. |

**Calibration order in Orca (PLA first):** temperature tower, then max volumetric speed, then flow ratio, then pressure advance, then retraction. Hardened steel usually needs about 5–10 °C more than brass.

**Cooling:**
- Two 5015 blowers move a lot of air. PLA probably needs about 50–80%, with 100% for overhangs and bridges.
- ASA needs the fan off for the first layers, then about 10–30%.
- To-do: find the speed below which the blowers stall, then set `off_below` and `kick_start_time` in `[fan]`.
- Watch for the nozzle temperature dropping when the fan comes on at layer 2. If it drops by more than about 2–3 °C, re-run the hotend PID tune with the fan on.

**ASA:** bed at about 95–105 °C, nozzle at about 250–260 °C. Needs an enclosure and ventilation because of styrene fumes. Check whether the bed holds 100 °C, and retune the bed PID at 100 °C if it doesn't.

---

## 7. Incidents and lessons learned

- **2026-10-22, nozzle crashed into the bed during G28.**
  - The accelerometer had been mounted *between the nozzle and the heater block* with two silicone washers. That lowered the nozzle about 3–4 mm below the BLTouch trigger point, so the nozzle hit the bed before the probe triggered.
  - The mainboard dropped off USB and needed a power cycle, a Klipper restart and a `FIRMWARE_RESTART`. No damage was found.
  - **Rule:** never home Z with the sensor on the nozzle. Either mount it on the toolhead above the nozzle tip, or home only X and Y and use `SET_KINEMATIC_POSITION Z=0`. That needs a temporary `[force_move] enable_force_move: True`; remove it afterwards.
  - After removing the nozzle, re-tighten it **hot** (about 240 °C), snug, holding the heater block with a second wrench. Then redo the Z-offset.
- **2026-10-22, backups had stopped since 2026-04-11.** There were two causes: a corrupt git object in `~/config_backup` (fixed by re-cloning) and an expired GitHub token (replaced).
- **2026-10-22, the GUI "Update All" failed.** Debian bullseye-security packages return 404 errors because they moved to `archive.debian.org`. Fixed by switching the source in `sources.list`.
- **The Pi's IP address changes after a reboot,** because it uses a dynamic DHCP lease. Reserve its address in the router.

---

## 8. Open to-do list

- [ ] Bed screws adjust and screws tilt calculate (user at the printer)
- [ ] `PROBE_ACCURACY`, then `PROBE_CALIBRATE` with the 0.10 mm gauge. **Required before any print.**
- [ ] Blower stall test, then set `[fan] off_below` and `kick_start_time`
- [ ] Orca: heat the bed before meshing (or use `START_PRINT`), turn off firmware retraction, set max volumetric speed to about 10
- [ ] Orca PLA calibration: temperature, max flow, flow ratio, PA, retraction
- [ ] ASA: bed holds 100 °C? ASA filament calibration. Enclosure?
- [ ] Reserve the Pi's address in DHCP
- [ ] Optional: centre `safe_z_home` for the probe; raise the webcam resolution in crowsnest

## 9. Done

- [x] Restored SSH access and the GitHub backups
- [x] Updated the OS packages and every Klipper component; flashed the mainboard to v0.13.0-816
- [x] Built and flashed the accelerometer firmware
- [x] KAMP and M600 fixes; narrowed Moonraker's trusted clients
- [x] PID-tuned the hotend and bed
- [x] Input shaping for X and Y; `max_accel` set to 4000
