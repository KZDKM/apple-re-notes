Apple Device Tree: iBoot's hardware description passed to the macOS kernel, `DeviceTree.j414sap.im4p`
- Decoded at `~/re/macos-13.5-22G74-j414s/firmware/payloads/DeviceTree.j414sap.json`, but the JSON mangles binary props -> `~/re/tools/adt.py` on the `.bin`
- `reg` addresses are relative to the parent's `ranges`, walk up to get a PA (arm-io: child 0 -> 0x200000000)
- `function-*` props: `<phandle> <4CC> <args>` = a platform function on another driver
	- `/cpus/cpuN`: `function-cpu_idle` pmgr `cpuI`, `function-enable_core` pmgr `Core` `BIT(4 * cluster + core)`, `function-error_handler` arm-io `ErrH`
	- `ane0` / `ave0`: `function-mcc_dataset` mcc `M$DS` (system cache streams)

## `/arm-io/pmgr`
- `reg[n]`: MMIO regions, n = what `mapDeviceMemoryWithIndex(n)` maps ([[AppleT6020PMGR/initRegMaps|initRegMaps]])
- `clusters`: 3 x (u16 core count, u16 0) = 4 / 4 / 4
- `soc-clusters`: 28 byte entries (phys ID, ?, flags, cluster ID, ..., voltage rail @11, name @12)
	- EACC phys 0 / id 1, PACC0 1 / 2, PACC1 2 / 3, ANE0 3 / 6 (rail 4), DISPLAY 4 / 8, AVEMSR 6 / 9 (rail 13)
- `ps-regs`: 12 byte entries `{reg index, offset, mask}`, PS reg = `reg[idx]` + offset + 8 * device `addr_offset`
	- `ps-regs[4]` = CPU cores + CPMs, `[6]` = `ANE_SYS` area, `[8]` = ANE sub domains `0x28e084000`
- `devices`: 411 x 48 byte records (m1n1 `struct pmgr_device`, dump with `adt_pmgr_devices.py`)
	- 0 flags (0x10 virtual, 0x04 no auto PM), 1 vote level (virtual voltage votes), 2 perf domain, 3 PMP / PTD id, 4 / 6 u16 parents, 8 u16 ?, 10 PS offset, 11 `ps-regs` index, 12 energy index ?, 15 auto min level, 24 bridge count, 25 gated cluster, 26 u16 device id, 28 `device-bridges` id, 32 name
- `device-bridges`: 19 x 76 bytes `{u8 pad[3]; u8 id; u8 bridge[4]; 68 zero bytes}`, bridge n = `reg[bridge-reg-index + n]`
	- `bridge-reg-index` = `0x30`, `#bridges` = 25, `bridge-settings-version` = 1
	- `ANE_SYS` -> id 5 -> bridge 4 -> `0x284000000`, SIO -> bridges 16 / 21 / 22
- Feature props (`pmp`, `*-tvm`, `soc-cluster-power-gating` ...): [[ApplePMGR/Features|Features]]
- `ane0` regs: engine `0x284000000`, pmgr `0x28e080000` (+0x4034), "set" `0x28e08c000` (+0x4000), `ane-type` 0xa0
- `/cpus`: cluster 0 = E (cpu0-3), 1 and 2 = P (cpu4-7, cpu8-11)
- `smc-pmu` `event_name-bitN`: MBSW wake bits (0 pwrbtn, 8 lid, 11 wifibt, 40 acattach, 48 rtc ...)
- Sleep related flags: `losesPowerInSleep` (clvr-spmi0..2), `shutdown-sleep` (ans, gfx), `manual-enable-s2r` (apcie bridges), `no-resume-restore` (gpio)
- No `sleep-pwr-policy` node on J414s -> `AppleSleepPowerPolicy` not loaded
