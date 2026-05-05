# BS21E SLE Ranging Pitfalls

## Directory and build identity
- The folder being edited may not be the folder being compiled.
- Treat the IDE/build output path as source of truth.
- Keep a visible `PROJECT_MEMORY.md` at the actual repo root.

## Generated config can override source defaults
- `Kconfig` defaults are not always what the final build uses.
- Check `build/config/target_config/bs21e/menuconfig/acore/*.config`.
- If runtime threshold still behaves as `1.00m`, inspect `CONFIG_MEASURE_DIS_DEFAULT_THRESHOLD_CM` in generated config.

## IQ pairing and stale data
- Local IQ and remote IQ arrive asynchronously.
- Do not run the algorithm against a global local IQ buffer that callbacks can mutate.
- Snapshot local IQ before calculation.
- Drop remote IQ while the algorithm is busy to avoid backlog and stale timestamp pairs.

## Queue backlog
- `remote iq queue backlog` means calculation is slower than incoming remote reports.
- Queueing more remote IQ usually makes results older, not better.
- Prefer dropping while busy for this single-result alarm use case.

## Timeout false positives
- Distance calculation can take around `800-1100 ms`.
- Indicator timeout must not fire while the algorithm is busy.
- Otherwise logs can show fallback to waiting state even though a calculation is still completing.

## Buzzer behavior
- Buzzer should follow current alarm state, not a hardcoded beep duration.
- Hysteresis is intentional: it prevents threshold-edge chatter.
- With center threshold `2.50m` and hysteresis `15cm`, alarm enters at `2.65m` and exits at `2.35m`.
- If the user asks for immediate stop at the center threshold, hysteresis must be reduced, but that can cause chatter.

## Filtering terms
- Vendor SLEM smoothing is inside `slem_alg_calc_smoothed_dis`.
- Application jump filter usually means `MEASURE_DIS_MAX_JUMP_CM`.
- Hysteresis is not smoothing; it only controls alarm enter/exit points.
- Do not add extra IIR/average smoothing unless the user explicitly accepts slower response.

## Build and flashing
- A Windows `WinError 32` during post-build move of `application.bin` usually means file lock, not bad C code.
- `application_sign.bin` is the normal direct flash artifact for this signed BS21E flow.
- `application_std.hex` is for tools that require hex.
- `fota.fwpkg` is for OTA/FOTA, not normal direct flashing.
- `application.bin` is raw unsigned output.

## Runtime logs
- `slem_alg_calc_smoothed_dis failed. ret:0x8000a453` can be a frame-level algorithm failure.
- One failed frame does not mean the task crashed if later frames continue.
- A reboot banner with exception count `0` is not enough to prove hardfault; check reboot cause and exception counters.

## Stack and compile issues seen
- Indicator task stack overflow was seen earlier; keep task stacks conservative.
- `g_stack_enable_result` should be wide enough for `errcode_t` values on the server side.
- Advertisement data lengths should match expected API width; `uint16_t` was used for server adv lengths.
