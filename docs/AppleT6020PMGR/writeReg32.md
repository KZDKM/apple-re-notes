T6020's register write wrapper, hides hardware workarounds behind plain PS writes
- `AppleT6020PMGR::writeReg32(RegMap, off, val, die)`, `0xfffffe0009b814f0`
- `ANE_SYS` (map 0 `0x260`) going off (new target 0, current nonzero): [[enableTVM]](4, off, wait) BEFORE the PS write
	- -> on macOS ANE TVM is off whenever `ANE_SYS` is off
	- Linux genpd writes `ane_sys` plainly, ANE TVM stays on (reads 1) -> every Linux ANE cut ran with TVM on
	- Turning it off by hand on Linux hangs the SoC ([[Cluster control]])
- Table of 10 fabric PS regs (map 0 `0x1f8` / `0x218` / `0x1f0`, map 2 `0x168` / `0x188` / `0x190`, map 3 `0x1a8` / `0x1b0` / `0x1c0` / `0x1c8`, all AFNC): going off -> set bit 29 in 1-2 companion regs, write, wait busy (bit 11) and actual (7:4) clear, clear bit 29
- Special cases: map 0 `0x258` (`DISPEXT0_SYS`) with map 6 `0x100`, map 0 `0x120` with map 2 `0x1c0`, map 6 `0x108` on t6022 rev <= A1
- After the write: wait busy clear for map 0 PS regs and some map 2 / 3 ranges
