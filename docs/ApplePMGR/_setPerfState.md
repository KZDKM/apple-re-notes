Sets a device perf domain's state, only used for the ANE on T6020
- `_setPerfState(state, u8 perfDomain, die)`, perf domain from device record byte 2 (`_deviceIDToPerfDomain`, < 0x30, 0 = none)
- Only domains 8 (ANE) and 14 (ANE1), anything else REQUIRE panics
	- 8 -> RegGroup 32 `ANE_PERF_STATE`, 14 -> 33 `ANE1_PERF_STATE`
- Wait bit 30 clear, write low nibble = state
- `ANE_PERF_STATE` = map 82 + `0x1200`, map 82 = ADT pmgr `reg[41]` = `0x285868000` -> PA `0x285869200` (inside the ANE window)
	- RegGroup 6 `ANE_TVM` = map 82 + `0x2000` = `0x28586a000`
- Linux: reads `80000000` = state 0 (only readable while `ANE_SYS` is on)
