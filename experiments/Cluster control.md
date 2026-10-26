`CLUSTER_CTL` = RegGroup 2 = map 0 + 0x8000, PA `0x28e088000`, one word per phys cluster ID ([[enableCluster]], [[getPhysicalClusterID]])
- Words: 0 EACC, 1 PACC0, 2 PACC1, 3 ANE0, 4 DISPLAY, 6 AVEMSR, 5 / 7 none (from ADT `soc-clusters`)
- Linux s2idle (smcpg `0x40`), words 0..7: `0f 2f 2f 2f 2f 00 2f 00` at suspend and resume, every cluster targeted on
	- Bit 5 set on all except EACC, meaning TBD
- macOS [[AppleT6020PMGR/quiesceHW|quiesceHW]] targets all of them off before S2R (IDs 1..9, wait only on non CPU)
- Target off around s2idle (smcpg `0x80`, `cluster_off_mask`, CPU clusters guarded)
	- AVEMSR: never wakes up (macOS also switches TVM for its rail first, we didn't)
	- ANE0: never wakes up, also with smcpg ordered after genpd (`run_last`) and `ane_cpu` / `ane_sys` already off
	- -> the write itself is fatal outside macOS's full sequence (hidden domains, TVM, PMP?), parked
