What runs on the NixOS install now and what's next (2026-10-26)

## Deployed (`~/nixos/hosts/macbook`)
- smcsleep, built from `~/re/smcsleep` with `--impure` (source stays out of the public repo), loaded at boot ([[Tools]])
- PMP boot ([[PMP dashboard]], [[Aurora-DeepIdle]])
	- `aurora/m1n1-pmp-headroom.patch`: m1n1 DT headroom 96K -> 160K, PMP tunable failure leaves PMP off instead of stopping the boot
	- `aurora/j414s-pmp.patch`: `#define APPLE_USE_PMP` in `t6020-j414s.dts`
	- Booted: node okay with 81 tunables, `apple_pmp` bound, "PMP started", one harmless unknown tunable warning
- `aurora/t602x-cpu-energy-model.patch`: EAS on (`sched_energy_aware` = 1)
- `power.nix`: CPU PD always (cpuidle `state0/disable` = 1 via tmpfiles)
- Reverted: ANE DT drop, `pmgr-pwrstate` probe gating patch

## Recovery
- `boot.bin` is one file on the NixOS ESP (`nvme0n1p4`), old generations share it
- Broken boot -> Fedora or macOS, mount the ESP, copy `/boot/m1n1/boot.bin.stock` over `boot.bin`

## Linux sleep state now
- CPU cores and all CPMs power gate in s2idle ([[CPU idle and clusters]])
- Fabric / DCS at p-state 1 in s2idle (pmgr-misc, PMP owns them now)
- Leftover clock gated domains gated by `pgate` ([[Domain inventory]])
- SMC told via slpw / waka ([[SMC sleep sequence]])
- Every SoC cluster stays on, cluster off blocked ([[Cluster gating]])

## Next
- Overnight runs: without smcsleep, with smcsleep, with `rfkill block all` (>= 6 h, battery, `power-log.sh`)
	- Compare with old 1.9 W, their 1.4 W, macOS 0.24 W
- PD always A/B awake with `pstr-avg.sh` (runtime toggle)
- Cluster off needs outside help
	- Bare m1n1 proxy: swap Fedora's ESP `m1n1/boot.bin` (`nvme0n1p6`) for plain `m1n1.bin` (no payload -> USB proxy), cut ANE from the proxy shell
		- Survives -> m1n1 / Linux boot path breaks it, diff PMGR dumps
		- Dies -> macOS sets something up at runtime
	- Hypervisor trace: guest must be macOS 13.5 / 14.x (newer XNU needs SPTM), so a separate 14.8.9 volume like [[Aurora-DeepIdle]]
