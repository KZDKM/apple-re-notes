PMGR feature table: 78 named switches filled from ADT pmgr properties (boot args override)
- At `this+800`, 16 byte entries `{name ptr; u8 present; u32 value}`, template `off_FFFFFE0007DA68E8`
- Interesting entries: 39 `pmp` -> `this+432` (2 on J414s), 56 `ane-dpe`, 62 `wake-time-events`, 66 `soc-cluster-power-gating` (`this+1868`, `0x2cc`), 67 `cpm-power-gating-hack`, 73 `acc-cluster-power-gating`, 21-36 `*-trig` (throttlers)
- `_enableFeature(f, en)` from [[AppleT6020PMGR/restoreHW|restoreHW]] (on) / `quiesceHW` (off)
	- Only 0-20, 33-35, 40, 41, 44-47, 49, 50 do anything: APSC, `*-tvm` ([[enableTVM]]), `dvmr`, throttlers, `l2c-*` (ACC path), PLL / APSC sleep, `cpu-scw`, `fane`
	- 66 / 67 / 73 fall to `default` -> pure software masks, no hardware enable
- Defined in the J414s ADT: `cpu-apsc` 1, `soc-tvm` 1, `disp-tvm` 1, `avemsr-tvm` 1, `dcs-tvm` 1, `fab-tvm` 1, `afr-tvm` 0, `pmp` 2, `soc-dpe` 1, `ane-dpe` 1, `apsc-snooze` 1, `acc-harvesting` 1, `soc-cluster-power-gating` `0x2cc`, `acc-cluster-power-gating` 1, `acc-notification-timeout` 5
- Throttlers: `enableThrottler(t, en)` ORs / clears the `*-trig` value into `*_TRIG_CTL` (RegGroups 10-18, ANE = 13 = `0x28e0f4000`), no `*-trig` props on J414s -> never changed
