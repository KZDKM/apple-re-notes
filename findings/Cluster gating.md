How macOS turns a SoC cluster off at runtime, and why Linux can't copy it yet
- Register: `CLUSTER_CTL` ([[Cluster control]]), the only writer is [[enableCluster]]
- `soc-cluster-power-gating` = `0x2cc` -> cluster IDs 2, 3, 6, 7, 9 gated at runtime, DISPLAY (8) never

## Peripheral clusters (ANE, AVE)
- One gate device per cluster, ADT device record byte 25 ([[ADT]]): `ANE_SYS` -> 6, `AVEMSR-V` (virtual) -> 9, `PCPM0/1` -> 2 / 3
- Off ([[_syncDeviceStatusChange]], devices in reverse)
	1. `_tearDownDeviceBridges`: only saves bridge counters
	2. Perf state 0 ([[_setPerfState]], only ANE perf domains 8 / 14)
	3. PS off through T6020 [[writeReg32]], which for `ANE_SYS` first turns ANE TVM off
	4. PMP dashboard bit cleared, wait for PMP ack ([[PMP dashboard]])
	5. Gate device -> hysteresis timer, `entry+10` = off pending
	6. Timer -> `_clusterHysteresisTimeoutHandlerGated` -> `enableCluster(id, 0, wait, die)`
- On
	1. Off pending -> cancel the timer, cluster never went off
	2. Else `enableCluster(id, 1, 0, die)`, sleep until the cluster reports F (cluster interrupt, `CLUSTER_CTL` bit 5 = status)
	3. PS on, `_initDeviceBridges` (`_configureBridge` + counters), perf state
- Also at boot: [[ApplePMGR/start|start]] -> [[AppleT6020PMGR/restoreHW|restoreHW]] -> `_syncDeviceStatus` gates every cluster whose gate device is off
- ANE / AVE drivers' own power off only undoes state from their power on (firmware, DARTs, mcache streams, internal sub blocks, AVE DPM / DPE)

## CPU clusters
- Only after the cores are shut down, see [[CPU idle and clusters]]

## Ruled out on Linux (all cuts still die)
- Live PS domains: every domain in ANE / AVE read 0/0
- Perf state: `ANE_PERF_STATE` = 0
- Bridges: teardown is counter bookkeeping, init only matters for a restore
- TVM on AVE rail: already off on Linux
- Gating enable bits: the masks are software only, `enablePowerDomainGating` is empty on T6020
- Throttle trigger `ANE_TRIG_CTL`: `ane-trig` absent, macOS never changes it
- PMP dashboard: PMP wasn't running, nothing reads it
- Fabric parent `AFNC0_LW0`: always on in macOS too, no isolation step exists
- Restore half: the clean awake cut died too
- `0xC000` bit 31 before the cut: still fatal
- ANE TVM off: fatal by itself, without and with PMP running

## Still open
- Some precondition outside what PMGR's code shows, maybe PMP dashboard + TVM + PS off back to back, or iBoot / m1n1 boot state
- Next: bare m1n1 proxy test, then a hypervisor trace with a macOS 14.x guest ([[Current setup]])
