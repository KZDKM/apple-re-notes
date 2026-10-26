Turns TVM (per voltage rail margin loop) on / off
- `AppleT6020PMGR::enableTVM(u8 rail, bool en, bool wait, u32 die)`, vtable `0xa08`, PAC disc `0xe748`
- Control bit 0 = enable, status bit 1 = busy (wait until clear when turning off)

| rail | block | control | status |
|---|---|---|---|
| 1 | SOC | PMGR `+0x4C000` | `+0x4C180` |
| 6 | DCS | `+0x4C200` | `+0x4C380` |
| 13 | AVEMSR | `+0x4C400` | `+0x4C580` |
| 9 | DISP | `+0x4C600` | `+0x4C780` |
| 10 | ? | `+0x4C800` | `+0x4C980` |
| 14 | FAB | SOC east `+0x30000` (`0x2902b0000`) | `+0x30180` |
| 4 | ANE | ANE clock gen `+0x2000` = `0x28586a000` | wait `+0x2008` bit 0 = 1 |
| 0 / 7 / 8 | CPU | ACC reg `0xE20048` | ACC `0xE20020` bit 6 |
| 2 / 3 / 15 | - | no op | |

- Callers: [[ApplePMGR/Features|_enableFeature]] (boot), [[enableCluster]], `restoreANE` (only with `ane-tvm`), T6020 [[writeReg32]] (ANE), perf state change paths, interrupt handler
- macOS boot: on for SOC / DCS / AVEMSR / DISP / FAB
- Linux reads: SOC / DCS / AVEMSR / DISP / FAB all off, ANE on
