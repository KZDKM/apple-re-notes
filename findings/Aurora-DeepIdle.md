Someone else's J414s idle / sleep work on linux-aurora, same machine
- Repo: https://github.com/JJRPF/Aurora-DeepIdle (start at `docs/status.md`)
- Measured with paired A/B runs on SMC power sensors, raw data in the repo

## Numbers
- macOS sleep on this machine: 0.24 W, 98.7% of it in S2R
- Their Linux s2idle: ~1.4 W, with or without the PMP
- Awake idle matches macOS once the PMP runs -> the gap is SoC sleep itself, not devices
- macOS keeps SD reader, TB NHI, ANE and external display pipelines off when nothing is plugged in

## Awake power reductions
- PMP boot: 0.9 - 1.15 W less awake
	- PMP = fabric / memory power controller, J414s never boots it
	- m1n1 copies 81 PMP tunables into the DT, needs ~16K but only ~13K of the 96K headroom is left -> boot stops
	- Fix: 160K headroom, plus leave the PMP off instead of failing if it still doesn't fit
	- DT side: `#define APPLE_USE_PMP` in the J414s dts, like J314s already does
- CPU PD always (disable cpuidle state0 WFI): ~1 W less on a busy desktop
	- `cpuidle-apple` only picks CPU PD for idles > 10 ms, busy desktop wakes ~1500/s per CPU -> mostly plain WFI
	- Cost 10-15 us wake latency, quiet fresh boot only gains 0.1 - 0.15 W
	- 100 us residency gets ~94% of it (their `pd_residency_us` patch), I went with PD always
	- Fits my cpuidle stats: PD and WFI each about half of idle time
- CPU energy model (`opp-microwatt` for T602x), EAS moves light work to E cores: ~60 mW

## Sleep side
- External DCPs cost nothing measurable with nothing attached -> dropped my dispext idea
- HDMI rails 0.13 W outside the SoC, DCP driver never lowers them (their patch untested on boot)
- SD reader link without ASPM L1 costs 0.37 W, PCI remove / rescan breaks it (and Wi-Fi)
- PMU RTC alarm driver: `rtcwake -m freeze` works, useful for unattended sleep tests
- SMC handshake same as mine ([[SMC sleep sequence]]): slpw harmless, slpa kills NVMe until full power off
- PMP sleep = RTKit IDLE (0x201), acks but never wakes again, macOS wakes it by restarting its CPU (TBD)

## Hypervisor
- macOS guest must be 13.5 or 14.x, newer XNU needs SPTM which m1n1 lacks -> my macOS 27 can't be traced
	- They use a separate macOS 14.8.9 volume
- Traced macOS sleep entry (SMC + PMP tracers): `MSxb` writes, MBSE slpw, PMP display targets down to only PCPM0 / PCPM1, MBSE slpa, `SetAPPower` 0x201 to SMC and PMP, CPUSTART, cores deep WFI
- Device power on macOS goes through the PMP dashboard (`DEV_STATUS_TGT_WR` request, `DEV_STATUS_ACT` ack), same as my PTD reading
- They didn't trace PMGR / CLUSTER_CTL

## Relation to my work
- My cluster off dead end ([[Cluster control]]): PMP not running on Linux is now the best explanation
	- ANE TVM off hangs, likely a voltage change nobody serves without PMP
	- Next: retry the ANE sequence with the PMP booted and `pmp-ane-sys` enabled
- Mine they don't have: CLUSTER_CTL / writeReg32 TVM wrapper, smcsleep pgate, fabric / DCS at 1/1 in s2idle, CPMs gating by themselves
- Their repo has unattended root harnesses (touch `/boot`, reboot), read only, don't run
