# BS21E SLE Ranging Log Patterns

## Normal frame

```text
RECEIVE LOCAL IQ. timestamp_sn:...
store local iq data complete.
RECEIVE REMOTE IQ. timestamp_sn:...
slem recv remote iq.
local ts = ..., remote ts = ..., local complete:1.
SLEM get distance done. distance:x.xxx, time = xxx ms.
measure_dis distance:x.xxm threshold:y.yym far:0/1 buzzer:0/1
```

Meaning: IQ pair matched, algorithm returned a valid distance, indicator state updated.

## Algorithm frame failure

```text
slem_alg_calc_smoothed_dis failed. ret:0x8000a453
REMOTE IQ CALC FAIL
MSG PROC ERROR type:1
```

Meaning: current frame failed. If later distance logs continue, this is not a crash by itself.

## Stale timestamp or wrong pair

```text
local:..., remote:...
MSG PROC ERROR type:1
```

Likely cause: remote IQ paired with an older/newer local IQ. Check snapshot and busy-drop behavior.

## Queue backlog

```text
remote iq queue backlog:1, drop.
```

Meaning: algorithm is slower than incoming remote IQ. For the alarm use case, dropping stale work is usually better than queueing it.

## Timeout fallback

```text
measure_dis distance timeout, fallback to waiting state.
```

Meaning: no valid distance for too long. This should not fire while the algorithm is busy.

## Reboot logs

```text
boot.
Flashboot Init!
Power On
Reboot cause:0xF0F0
exception reboot count:0
```

Meaning: not enough to prove a hardfault. Check exception count, reboot cause, and whether runtime logs stopped before reboot.

