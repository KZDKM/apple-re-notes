Fills `regGroups[group]`: present = 1, map, offset
- Called per group by `AppleT6020PMGR::initRegGroups`
- `getRegGroupOffset(group)` reads it back
- J414s: `CLUSTER_CTL` (group 2) = map 0 + 0x8000 -> PA `0x28e088000`
