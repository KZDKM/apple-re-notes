Decides if 12V main can be turned off during sleep, true if total sleep power < budget
- USB devices: sleep / remote wake power per device (`determineUSBPower`)
- PD ports: reads `DJ1D`..`DJeD` (14 ports), adds PD power of type 3 ports
- PCIe: reads `MIPO` (2 bytes), bit 15 valid, popcount of the rest x 1237.5 mW
- Budget: ADT `standby-budget` - 2500 mW, read in `AppleSleepPowerPolicy::start`
	- Zero budget -> driver stops
