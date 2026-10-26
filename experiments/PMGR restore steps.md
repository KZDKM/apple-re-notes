Replaying [[AppleT6020PMGR/restoreHW|restoreHW]] step by step in smcpg `resume_noirq`
- Register maps / groups first: [[initRegMap]], [[initRegGroup]], [[ADT]]
- `0x10`: step 1, PMGR `0xC000` bit 31
	- Display never comes back, on plain s2idle and with slpa (`0x3d`)
	- Not write 1 to clear: still 0 after a real slpa
	- -> needs the matching quiesce or the rest of the restore
- `0x20`: step 2, SEP force on (`PS_MIN` = 0xF, wait actual 0xF, clear) works on plain s2idle
	- SEP actual 0 -> F -> back to auto gating (0x4)
	- `regmap_update_bits` also clears the write 1 to clear `WAS_*GATED` flags, macOS does the same
