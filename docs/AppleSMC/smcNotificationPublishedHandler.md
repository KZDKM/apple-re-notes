Runs when root domain is published, registers PM settings with `IOPMrootDomain::registerPMSettingController`
- SMC has key `CLKH` -> time zone setting -> `smcSetTimeCallback` writes CLKH
- SMC has key `WKTP` -> `this+0x170` = `OSSymbol("SilentRunning")`
	- `SilentRunning` setting -> `smcSilentRunningCb` writes WKTP = setting value
	- Also gates WKTP = 2 in `AppleSMC::`[[AppleSMC/setPowerState|setPowerState]]
