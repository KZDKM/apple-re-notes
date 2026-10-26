PMGR Active action after S2R, and also every macOS boot ([[ApplePMGR/start|start]]), in order
1. Per die: PMGR map 0 `0xC000` |= bit 31
	- Linux reads 0 (`0xC004..0xC038` = `80000000`), only writer is this function
	- Setting it on Linux kills the display, and doesn't make the cluster cut survivable ([[Cluster control]])
2. Hibernating only: `updateHibDeviceStatus`, `notifyPMPInitialDeviceStatus`
3. SEP force on: SOC east `0xC00` (= Linux `power-controller@c00`, label `sep`, always on)
	- Set `PS_MIN` (bits 19:16) = 0xF, wait `PS_ACTUAL` (7:4) == 0xF, clear `PS_MIN`
	- Works on Linux s2idle (smcpg `0x20`)
4. `restoreSOCDPE`
5. `ApplePMGR::restoreHW`
	- `enablePowerDomainGating(1)` (empty on T6020)
	- `_syncDeviceStatus`: clusters on / off from gate device states ([[Cluster gating]])
	- `setWakeTimeFromSleep`, perf counters, `restoreACC` per CPU complex, energy counters, `_restoreSOCDPECounters`
	- `enableACG(1)`, `enableVoltageChanges(1)` (only a panic check on T6020)
	- `_enableFeature` for every valid feature ([[ApplePMGR/Features|Features]]), `_applyParam` 0-24
6. `enableCPMPowerDomainGating`: bit 31 of map 0 `0xC030` / `0xC034` / `0xC038`, already set on Linux
7. `restoreClockConfig`
8. `_setFABPerfState(5)`, `_setDCSPerfState(7)` -> `(9)`
