PMGR Quiesce action right before the S2R power off
1. `ApplePMGR::quiesceHW` first
	- Disable + snapshot perf / energy counters, timestamp
	- Features 0x4D..0: `_enableFeature(f, false)` if valid and not excluded (reversed by `_enableFeature` in restore)
	- Device perf state check loop, only logs
2. Per die: if mini PMGR (map 1) `0x4D010` bits 18-19 not both set -> map 6 `0x100` = 0, wait bits 7:4 == 0 (power gates a domain)
3. chip ID 0x6022 only: map 3 `0x288` / `0x290` off
4. Not hibernating: [[enableCluster]](id, false, id > 3, die) for cluster IDs 1..9 -> every SoC cluster targeted off
	- CPU clusters (IDs 1-3) not waited on, ANE / display / AVE waited on
