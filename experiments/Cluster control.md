`CLUSTER_CTL` = RegGroup 2 = map 0 + 0x8000, PA `0x28e088000`, one word per phys cluster ID ([[enableCluster]], [[getPhysicalClusterID]])
- Words: 0 EACC, 1 PACC0, 2 PACC1, 3 ANE0, 4 DISPLAY, 6 AVEMSR, 5 / 7 none (from ADT `soc-clusters`)
	- Bits 3:0 target, bit 4 busy, bit 5 cluster interrupt status (write 1 to clear, `clearClusterInterruptStatus`)
- Linux s2idle (smcpg `0x40`), words 0..7: `0f 2f 2f 2f 2f 00 2f 00` at suspend and resume, every cluster targeted on
	- `0x2f` = a status pending since boot that nobody clears
- macOS [[AppleT6020PMGR/quiesceHW|quiesceHW]] targets all of them off before S2R, at runtime see [[Cluster gating]]

## Attempts, all fatal
- AVEMSR off around s2idle (smcpg `0x80`): never wakes
- ANE0 off around s2idle, also with `run_last` and `ane_cpu` / `ane_sys` already off: never wakes
- PACC1 off on a `maxcpus=4` boot, cores parked in m1n1's `wfe` loop: instant SoC hang
	- macOS only cuts a CPU cluster after deep sleeping the cores and `quiesceACC` ([[CPU idle and clusters]])
- ANE0 off awake with ANE + DARTs dropped from the DT, all 9 ANE domains 0/0: instant hang, watchdog reboot after 5-10 s
	- -> the cut itself is fatal, not our old restore
- `0x200`: `0xC000` bit 31 then ANE cut in s2idle: never wakes
- `0x400`: ANE TVM off in suspend, ANE cut in `suspend_noirq`: machine powered itself off (clean power off, not watchdog)
- ANE TVM off alone, awake (`ane_tvm`): instant hang + watchdog reboot
	- Same with the PMP booted ([[Current setup]])
- Don't rerun `ane_tvm 0`, `0x200` or `0x400` without new information
