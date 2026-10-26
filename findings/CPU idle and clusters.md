What CPU cores and clusters do in Linux idle / s2idle, and how macOS takes a CPU cluster off

## Linux idle states
- `cpuidle-apple`: WFI (clock gated) and CPU PD (`CYC_OVRD` WFI_MODE=3, retention on, core power gated, wakes in place)
	- CPU PD target residency 10 ms -> busy desktop mostly WFI, PD always saves ~1 W ([[Aurora-DeepIdle]], applied)
- `ACC_OVRD` never touched by Linux, m1n1 deep sleep sets it + `DISABLE_WFI_RET` (ends at RVBAR)
- s2idle uses the deepest state anyway

## Core / CPM power registers
- PMGR `ps-regs[4]` = `0x28e080000 + 8n`: `ECPU0_0..3`, `PCPU0_0..3`, `PCPU1_0..3`, `ECPM` `0x60`, `PCPM0` `0x68`, `PCPM1` `0x70`, not in Linux's DT
- Target 0 on all, even busy ones -> target isn't the on / off request for CPUs, hardware decides
- Sampling awake (smcpg `cpups`, 100k samples): idle cores actual 0 83-97% of the time, `PCPM0` 0 for 38%
	- -> a cluster's CPM power gates by itself once all its cores are in PD
- s2idle (`suspend_noirq`): every core and CPM at 0 except the core running the callback
- -> CPU side of s2idle is near its floor short of a real cluster off, P cluster work parked

## macOS idle path
- ADT `/cpus/cpuN`: `function-cpu_idle` = pmgr `cpuI`, `function-enable_core` = pmgr `Core` arg `BIT(4 * cluster + core)`
- `AppleARMCPU::_processorIdle` -> `ApplePMGR::_cpuIdle`: per complex idle count / mask
	- Last core idle / first awake -> `cpuComplexIdleEnter` / `Exit` (T6020: only poll ACC reg `0xE20020` bit 31)
	- PMP doorbell only if PMGR feature `pmp` == 1, J414s ADT has 2 -> never
- `_cpuIdleInit`: only idle timer + `powergate_latency` (param 0) for xnu, no register writes
- Idle never turns a cluster off

## macOS CPU cluster off
- Only after every core of the cluster is shut down (processor shutdown, not idle)
	1. Per core `AppleARMCPU::quiesceCPU`: `enable_core(0)` (PMGR `enableCPUCores`) -> `ml_arm_sleep` (deep sleep, wake via reset vector)
	2. xnu `PE_cpu_power_disable` (stripped, kprintf names it) -> `disableCPUCluster` -> `quiesceACC` (+ ACC notification to PMP when `pmp` = 2) -> `CLUSTER_CTL` off
	3. Back: `PE_cpu_power_enable` -> `enableCPUCluster` -> `restoreACC`, cores restart through RVBAR
- Same order as m1n1's `smp_stop_cpu` + `cpu_sleep(deep)`
- On Linux: needs a P core restart path (m1n1 entry -> Linux), E cluster stays up so it's easier than full S2R, not started
- `maxcpus=4` + `CLUSTER_CTL` P cluster off from smcpg: instant SoC hang -> [[Cluster control]]
