After slpa the SMC really powers down parts of the SoC, macOS restores them in platform actions
- ADT `losesPowerInSleep`: `clvr-spmi0..2` (SPMI controllers of the core voltage regulator PMUs, `mirabeau`)
	- `AppleSPMIController::callPlatformFunction(gIOPlatformWakeActionKey)` rewrites 9 saved controller registers when set
- Others: `shutdown-sleep` (ans, gfx), `manual-enable-s2r` (apcie bridges), `no-resume-restore` (aop / nub / smc gpio)
- Two layers, each undoes its own sleep step
	- SMC: slpa / waka ([[SMC sleep sequence]])
	- PMGR: [[AppleT6020PMGR/quiesceHW|quiesceHW]] before power off, [[AppleT6020PMGR/restoreHW|restoreHW]] after
- slpa on Linux = redo every macOS platform action for blocks that lose state, list in [[Platform actions]]
