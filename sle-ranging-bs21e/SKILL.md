---
name: sle-ranging-bs21e
description: Use when developing or debugging BearPi/HiSilicon BS21E 星闪 SLE ranging firmware with IQ pairing, SLEM distance calculation, UART threshold updates, LED/buzzer alarm logic, build artifacts, flashing choices, queue backlog, stack overflow, timeout, and distance stability issues.
---

# SLE Ranging BS21E

Use this skill for BearPi/HiSilicon BS21E 星闪测距 projects, especially `application/samples/products/sle_measure_dis`.

## Start Here

1. Identify the actual build repo, not a mirror or copied sample folder.
2. Read `PROJECT_MEMORY.md` if present.
3. Check generated config under `build/config/target_config/bs21e/menuconfig/...` before trusting source defaults.
4. Separate these layers before changing behavior:
   - SLEM vendor algorithm smoothing
   - local/remote IQ pairing
   - application jump filter
   - threshold hysteresis
   - LED/buzzer indicator task timing

## Core Checks

- Search for `CONFIG_MEASURE_DIS_DEFAULT_THRESHOLD_CM`, `MEASURE_DIS_THRESHOLD_HYSTERESIS_CM`, and `MEASURE_DIS_MAX_JUMP_CM`.
- Search for `g_measure_dis_alg_busy` to confirm stale remote IQ is blocked while calculation is running.
- Confirm remote IQ processing copies a local IQ snapshot before calling the algorithm.
- Confirm timeout fallback checks `!measure_dis_alg_is_busy()` before changing indicator state.
- Confirm default flashing artifact from the standard target is `application_sign.bin` unless the tool explicitly requires hex or OTA.

## Common Fix Pattern

For unstable distance or `MSG PROC ERROR type:1`:
1. Do not calculate directly on mutable global local IQ data.
2. Copy a local IQ snapshot under a short IRQ lock.
3. Clear the local complete flag only after snapshot copy.
4. Mark algorithm busy before long calculation.
5. Drop new remote IQ while busy or when queue backlog exists.
6. Update indicator only after a valid distance result.

For buzzer complaints:
1. Check whether the user expects center threshold or hysteresis threshold.
2. Explain that a `2.50m` threshold with `15cm` hysteresis alarms at `2.65m` and stops at `2.35m`.
3. Do not add fixed buzzer duration unless requested.
4. Buzzer reaction is limited by the next valid ranging result.

## Reference Files

- Read `references/pitfalls.md` when diagnosing crashes, queue backlog, stale timestamp, file-lock build errors, or wrong flashing artifacts.
- Read `references/bs21e-sle-measure-dis-memory.md` when taking over the exact project state from the 2.5m buzzer threshold work.
