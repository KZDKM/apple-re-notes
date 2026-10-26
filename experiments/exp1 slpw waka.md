MBSE = slpw before suspend, waka after resume, both writes ok
- After resume MBSE = `30070000` (byte 1 0 -> 7, the wake reason macOS reports), MBSW cleared to 0
- SMC reacts to slpw, MBSE reads back a status not the written value
- waka doesn't reset it, still `30070000`
- Kernel never decodes byte 1, only reports it raw as wake reason `SMC.MBSE.07`, meaning of 7 TBD (powerd?)
