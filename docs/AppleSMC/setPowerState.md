`AppleSMC::setPowerState`writes key `WKTP` with value 2 on sleep 
- Calls `AppleSMCFamily::`[[writeKeyHostEndian]]
- Only writes if `this+0x170` is set
	- `this+0x170` = `OSSymbol("SilentRunning")`, set in [[smcNotificationPublishedHandler]] if SMC has key `WKTP`
- Check if this is what we need to do on suspend on linux.