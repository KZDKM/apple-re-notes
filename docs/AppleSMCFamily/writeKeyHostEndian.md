Writes key given a key name and value
- This is implemented in `macsmc` as `apple_smc_write`, no extra work needed
- Calls `AppleSMC::smcGetKeyInfo`, byte swaps 2-8 byte values, then `AppleSMC::smcWriteKeyWithSMC`
	- Message: byte 0 = 0x11 (write), byte 2 = size, high 32 bits = key, same as `macsmc`
