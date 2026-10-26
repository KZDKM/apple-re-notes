PMGR Active action after S2R, in order
1. Per die: PMGR map 0 `0xC000` |= bit 31
	- Linux reads 0 (`0xC004..0xC038` = `80000000`), only writer is this function
	- Setting it on Linux kills resume, with or without slpa
2. SEP force on: SOC east `0xC00` (= Linux `power-controller@c00`, label `sep`, always on)
	- Set `PS_MIN` (bits 19:16) = 0xF, wait `PS_ACTUAL` (7:4) == 0xF, clear `PS_MIN`
	- Works on Linux s2idle (smcpg `0x20`), SEP goes actual 0 -> F -> back to auto gating
3. `restoreSOCDPE`
4. `ApplePMGR::restoreHW`: `_syncDeviceStatus`, `_setPSRunMaxAll`, `_applyParam`, `_enableFeature`, counters
5. `enableCPMPowerDomainGating(this+0x36998)`
6. `restoreClockConfig`
7. `_setFABPerfState(5)`, `_setDCSPerfState(7)` -> `(9)`
- TODO: pair each step with [[AppleT6020PMGR/quiesceHW|quiesceHW]]
