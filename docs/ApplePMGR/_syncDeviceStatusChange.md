Applies a batch of device power state changes, and arms / cancels cluster gating
- `_syncDeviceStatusChange(DeviceStatusChangeData *, u32 count)`, input records `{u16 deviceID; u8 newPSLevel; ...; u32 die}` (8 bytes)
- Direction from the first non virtual device: new level below current -> off path (reverse), else on path

## Off path, per device
1. `_acquireDeviceLocks`, if it was F: `_tearDownDeviceBridges` (saves bridge counters only), save energy counters
2. Perf domain flag 0x10 and was F: `_setPerfState(0, perfdom)` ([[_setPerfState]])
3. `_setPSLevel(new)` -> PS write through T6020 [[writeReg32]]
4. Device has a gated cluster (record byte 25, bit in `soc-cluster-power-gating`)
	- No timer -> [[enableCluster]] now
	- Else `timer->setTimeout(entry+24)`, `entry+10` = 1 (off pending)
	- Fires -> `_clusterHysteresisTimeoutHandler` (finds entry by timer) -> command gate -> `...Gated` -> `enableCluster(id, 0, 1, entry+40)`, clears `+10`

## On path
1. Per gated cluster: `entry+9` = 1
2. Off pending -> `cancelTimeout`, clear `+10`; else `enableCluster(id, 1, 0, die)`, `commandSleep(entry+9)` unless state already F
3. Per device: `_setPSAutoMinLevel` (auto PM unless flag 4), `_setPSLevel`, if F: energy counters, `_initDeviceBridges` (`_configureBridge` + counters), perf state
4. Clear `entry+9`, `commandWakeup`

- `clusterInfo` = 48 byte entries ([[ApplePMGR/Class|Class]]): soc-clusters ptr +0, state +8, on in progress +9, off pending +10, timer +16, timeout +24, reporter +32, die +40
