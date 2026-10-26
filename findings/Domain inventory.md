Which PMGR power domains stay on through Linux s2idle (smcpg DT walk, `run_last`, 2026-10-24)
- 195 `apple,pmgr-pwrstate` domains in 4 PMGR blocks (`28e` 68, `290` 106, `29e` 18, `404` 3), 80 `apple,always-on`
- At `suspend_noirq`: 99 still on, 74 of them always on

## Buckets
- Always on in DT: DRAM / fabric (`dcs_*`, `amcc*`, `afnc*`, `soc_dpe`), `aic`, `pms*`, SPMI, `sep`, also `atc*_pcie`, `dptx_phy_ps`, `aes`, `pmp`, `debug`
- Held by a runtime active consumer
	- `disp0_*` (internal DCP, lid closed), `dispext0/1_*` (external DCPs, nothing attached)
	- `sio` / `sio_cpu` (SIO mbox + DART), `apcie_gp*` (Wi-Fi / SD), `atc0/1_usb` (dwc3 on ports 0 / 1)
	- `afr` (parent of `gfx`)
- No consumer, genpd thinks off, hardware at 4/4 (clock gated): `dwi`, `trace_fab`, `fpwm1/2`, `i2c5/6/7`, `sio_spmi0/1/2`
	- `pmgr-pwrstate` only counts actual F as active, the bootloader left them at 4 -> genpd never calls `power_off`
	- smcpg `0x100` / smcsleep `pgate`: target 0 in suspend, back to 4 on resume, all 10 reach 0, clean resume
	- A `pmgr-pwrstate` patch to gate them at probe was written then reverted (risk for a few mW)

## External DCPs
- Linux keeps `dispext*` on via apple-dcp
- macOS too while awake: IOReport `SoC Stats` / `Device States` shows `DISPEXT*` ACT 100% with or without a display ([[Tools]] `ioreport_states.c`)
- [[Aurora-DeepIdle]]: no measurable cost with nothing attached -> dropped

## ANE
- `ane_sys` / `ane_cpu` on while awake: its 3 DARTs are runtime active forever (no runtime PM, no unbind, `suppress_bind_attrs`)
- Off in s2idle (genpd), also the 7 sub domains (`ane_sys_mpm`, `ane_td`, `ane_base`, `ane_set1..4`, in the DT at `0x28e084000..30`) read 0/0
- DT drop of ANE + DARTs keeps them off awake too, tried for [[Cluster control]], reverted
