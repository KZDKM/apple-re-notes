PMGR init at every macOS boot, Linux runs none of it
- `ApplePMGR::start`: `initDriver` -> `initAON` -> `_initPMPv2` -> [[AppleT6020PMGR/restoreHW|restoreHW]] -> `lateRestoreHW` -> DVD factors -> `initFixedFreq`
	- `initAON`: AON perf counters, `setAPWakeupSources`
	- `lateRestoreHW`: `lateRestoreACC` per CPU complex, `enableThrottler` for present `*-trig` features (none on J414s), ROSC reads
- So `restoreHW` (incl. `0xC000` bit 31) runs at boot, not only after S2R

## Platform actions (`callPlatformFunction`)
- Sleep: state 6, `_quiescePerfStateChanges`
- Quiesce: state 4, [[AppleT6020PMGR/quiesceHW|quiesceHW]] (vtable `0x900`, disc `0xb183`)
	- xnu runs Quiesce actions from the boot CPU's `quiesceCPU` right before `ml_arm_sleep` -> every S2R
- Active: state 2, optional `boostPerfState`, `restoreHW` (`0x908`, disc `0x8e8f`), state 1
- Wake: DVD factors, fixed freq, `_processAPWakeSources`, `lateRestoreHW`
- HaltRestart: `haltRestartHW` (also calls `quiesceHW`)
