Writes MMPP (1 byte) = `!`[[turnOffPS]]`()` on power state 0 through `AppleSMCInterface::`[[smcWriteKey]]
- MMPP = 1 keeps 12V main on during sleep, 0 turns it off
- Driver matches ADT node `sleep-pwr-policy`, J414s doesn't have it -> never loaded, MMPP never written
