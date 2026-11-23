# Custom ArduCopter 4.7.3 forward port

**Status: source integration only; NOT built or flight-tested.**

Source branches:

- `dev/rssi-and-vtx`: original base, including the 1600/2000 mW VTX configuration and enabled extended RC-link OSD fields (commit `caa939d`, authored by `snktshrma`).
- `foxeer-tramp-reaper-fix`: protocol and power-handling fixes, selected as the source of this production-oriented port.
- `foxeer-tramp-reaper-diagnostic`: extra packet/baud logging, deliberately not incorporated into the production code.
- `foxeer-tramp-reaper-offline-autobaud-diagnostic`: diagnostic successor, also contains `Tools/foxeer_tramp_offline_autobaud.patch`. **That patch is not applied to this port.**

This port applies the `foxeer-tramp-reaper-fix` versions of five files on top of official `Copter-4.7.3`, preserving the upstream 4.7.3 dbm conversion optimization in `AP_VideoTX.cpp` (`power * 0.1f` instead of `power / 10.0f`).

## Files and features

- `libraries/AP_OSD/AP_OSD_config.h`: enable `AP_OSD_LINK_STATS_EXTENSIONS_ENABLED`.
- `libraries/AP_VideoTX/AP_VideoTX.h`: 12 power table slots, Tramp-leading-byte option (bit 8), getter for maximum VTX power and actual measured power.
- `libraries/AP_VideoTX/AP_VideoTX.cpp`: 1600/2000 mW entries and metadata range; arbitrary non-standard reported Tramp power values; Tramp-specific power update allowance.
- `libraries/AP_VideoTX/AP_Tramp.h` and `.cpp`: Tramp response parsing workaround, actual-power reporting, high-actual-power warnings, configuration retry behavior.

Changes to the old code made specifically during the forward port: initialize `_act_power_warned`, `_last_conf_freq`, `_last_conf_power`, `_last_conf_options`, and `_power_warn_pending` explicitly.

## Not yet verified

1. Compile for the actual flight-controller target. Example: `./waf configure --board MatekH743 && ./waf copter`. Verify the correct target and custom hwdef first.
2. Confirm firmware size and that OSD/Tramp are enabled on the target. No hardware support can be assumed merely from a successful build.
3. Bench-test with propellers removed: Tramp VTX startup, 9600-baud response handling, optional `VTX_OPTIONS` bit 8 (value 256), settings update and retry behavior, pitmode, and 1600/2000 mW selections.
4. Check reported and requested VTX power separately; VTX telemetry power is not independently verified RF output. Confirm maximum permitted RF power for the hardware and jurisdiction.
5. Check analog OSD extended link fields using the intended receiver. Confirm parameters survive migration from 4.6.x to 4.7.3.
6. If the Foxeer VTX remains unresponsive, use the saved diagnostic branch or explicitly review the offline-autobaud patch. Do not silently enable continuous GCS debug traffic in release firmware.
7. Review all other hardware-specific customizations, scripts, and local unpushed changes: these five committed files do **not** prove that every feature on the currently installed aircraft firmware is represented in GitHub.

Keep the 4.6.3 working firmware, `.param` backup, and old branches intact until both build and bench testing are complete.
