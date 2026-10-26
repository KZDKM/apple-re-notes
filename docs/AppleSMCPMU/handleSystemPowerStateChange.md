Handles system power messages, writes MBSE through `AppleSMCInterface::`[[smcWriteKey]]
- Registered in `AppleSMCPMU::start` via `registerPrioritySleepWakeInterest`, only if SMC has key `MBSE` (size 4)
- `kIOMessageSystemWillPowerOff`: MBSE=offw
- `kIOMessageSystemWillRestart`: MBSE=rest
- `kIOMessageSystemCapabilityChange` with CPU capability going away (sleep): MBSE=slpw
