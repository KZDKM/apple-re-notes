Fills `regMaps[map][die]` for one register region
- `initRegMap(map, id, die, optional)`
	- die 0: `provider->mapDeviceMemoryWithIndex(id)` -> `id` = ADT pmgr `reg` index
	- die > 0: die 0 range + per die stride
	- `optional`: skip silently if the die / reg doesn't exist, else panic
- Callers per SoC: `AppleT6020PMGR::`[[AppleT6020PMGR/initRegMaps|initRegMaps]]
