- `~/ida-tools` (README inside): `fieldx` (field access index), `ida_reanalyze` (`type_this`, `decomp_seg`), `ida_field` (`fld`), `ida_ipsw_vtables`, `ida_vcall`, `idah2bn`
- `~/re/headless/kc.i64`: copy of the IDA DB for idalib, `~/re/tools/hl/` scans
	- `decomp.py` (Hex-Rays by name), `opscan.py` (operand values), `vtdump.py` (vtables -> `vt_<Class>.tsv`), `smc_keys.py`
	- Finding vcall sites: read the PAC discriminator from the vtable's chained fixup (`ipsw macho dump <kc> <vt+off>`, bits 32-47), then `opscan` for `MOVK X17, #disc` -> exact callers, offset alone matches half the kernel
- `~/re/tools/adt.py`: raw ADT parser, decodes `function-*` props (the JSON mangles them)
- `~/re/tools/adt_pmgr_devices.py [adt] [name]`: ADT pmgr `devices` as a table, field docs in its header ([[ADT]])
- `~/re/tools/ioreport_states.c`: macOS only, PMGR `SoC Stats` device / cluster state residency via `libIOReport`

## smcpg
`~/re/playground/smcpg`, playground module, read only unless `exp=` / `allow_write=1`
- `exp=` bitmask
	- 1 slpw / waka, 2 WKTP, 4 slpa (polled, `suspend_noirq`), 8 early MBSE read + waka (`resume_noirq`)
	- 0x10 PMGR `0xC000` bit 31, 0x20 SEP force on, 0x40 dump `CLUSTER_CTL`, 0x80 cluster off (`cluster_off_mask`)
	- 0x100 power gate leftover clock gated domains ([[Domain inventory]])
	- 0x200 `0xC000` bit 31 then ANE cluster cut, 0x400 ANE TVM off then ANE cluster cut ([[Cluster control]])
- `run_last` (default on): device links so its noirq hooks run after genpd powered domains off / before it powers them on
- debugfs `/sys/kernel/debug/smcpg/`
	- `key` / `info` / `value` / `keys`: SMC keys
	- `domains`: every `apple,pmgr-pwrstate` domain (block, label, `*` = always on, target, actual)
	- `cpups`: CPU core / CPM / hidden ANE / AVE PS regs, fabric / DCS p-state, PMP PTD, TVM per rail, `ANE_PERF_STATE`, `ANE_TRIG_CTL`
		- `echo N > cpups`: sample N times 10 us apart, shows how often each actual state was seen
	- `cluster_off`: `CLUSTER_CTL` words, `echo 1/2` P cluster off (maxcpus=4 only), `echo 3` ANE0 (all ANE domains off)
	- `ane_tvm`: ANE TVM on / off by hand (only while `ANE_SYS` is on)
- `resume_hold_ms` + `console_suspend=N`: hold screen with log replay, for resumes that never reach the journal
- `KEY_POWER` dropped for 2 s after resume (late wake press otherwise resuspends)
- `kdir.sh`: finds a dev tree matching the booted kernel, keeps a GC root
- `pstr-avg.sh [secs] [tag]`: averages SMC power sensors (total, AC in, 3.8 V, heatpipe = SoC) into `pstr-avg.tsv`
- `power-log.sh`: `energy_now` before / after a sleep, average W

## smcsleep
`~/re/smcsleep`, the "usable" module, only proven tricks, loaded at boot on NixOS ([[Current setup]])
- MBSE slpw before suspend, waka after resume (also after an aborted suspend)
- `pgate` (default on): power gate the 10 leftover clock gated domains in `suspend_noirq`, back to 4 in `resume_noirq`
- `pwrkey_ignore_ms` (2000), `wktp` (off)
- Attaches through a platform bus notifier (macsmc is builtin), inactive without a 4 byte MBSE
- Don't load together with smcpg experiments (both write MBSE)
