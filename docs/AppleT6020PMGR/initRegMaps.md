`initRegMap(this, map, id, 0, 0)` for 48 maps, `id` = ADT pmgr `reg` index
- map 0 -> `reg[0]` = PA `0x28e080000` (main PMGR, Linux `power-management@28e080000`, Linux maps only 0x0..0xC000)
- map 1 -> `reg[1]` ("mini PMGR")
- map 2 -> `reg[2]` = PA `0x290280000` (SOC east, Linux `power-management@290280000`)
- map 6 -> `reg[44]`
- PA = `reg` address + arm-io `ranges` (child 0 -> 0x200000000)
