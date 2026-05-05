# Handoff Playbook

## First prompt for a new Codex

```text
请先阅读本仓库根目录的 AGENTS.md、PROJECT_MEMORY.md、docs/DEBUG_PLAYBOOK.md。
同时使用 c-skill 仓库里的 sle-ranging-bs21e skill。
这是 BearPi/HiSilicon BS21E 星闪 SLE 测距项目，当前重点是 server 侧测距、UART 阈值修改、LED/蜂鸣器报警。
不要从零猜业务逻辑，先确认实际编译目录、生成配置、当前参数和烧录文件。
```

## Questions to answer before editing

1. Which directory is actually being built?
2. Is the uploaded repo a full SDK or only the sample slice?
3. Does generated menuconfig override the Kconfig/source default?
4. Is the user asking about threshold center, hysteresis, jump filtering, or algorithm smoothing?
5. Is the symptom from code behavior or from measurement update cadence?

## Minimum files to inspect

- `PROJECT_MEMORY.md`
- `docs/DEBUG_PLAYBOOK.md`
- `application/samples/products/sle_measure_dis/Kconfig`
- `application/samples/products/sle_measure_dis/sle_measure_dis_server/sle_measure_dis_server.c`
- `application/samples/products/sle_measure_dis/sle_measure_dis_server/sle_measure_dis_server_alg.c`
- `build/config/target_config/bs21e/menuconfig/acore/*.config` in the full SDK tree, if available

## Preserve these decisions

- Keep local IQ snapshot before long algorithm calculation.
- Keep algorithm busy flag.
- Drop remote IQ while busy or when backlog exists.
- Keep timeout from forcing waiting state while algorithm is busy.
- Keep jump filter at `300cm` unless the user changes that explicitly.
- Do not re-add extra averaging/IIR smoothing without explicit consent.

## Explain these plainly

- `threshold` in logs is center threshold.
- `far` is alarm state after hysteresis, not just `distance > threshold`.
- `buzzer` follows `far` and valid connection/distance state.
- `application_sign.bin` is the normal signed direct-flash artifact.

