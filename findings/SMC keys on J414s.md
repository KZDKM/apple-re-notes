Read with smcpg `info` / `value` (fw 27.0)
- WKTP: `ui8`, size 1, flags 0xd0 (r w function), 00 while awake
- MBSE `hex_` size 4, MBSW `hex_` size 8, both flags 0xd4 (r w function, 0x04 = raw bytes, no byte swap)
- MBSE value is the little endian u32 of the 4CC: slpw = `77706c73`, slpa = `61706c73`, waka = `616b6177`
- Awake: MBSE = `30000000`, MBSW = `0100000000000000`
	- MBSW is a 64 bit wake event bitmap, bit 0 = pwrbtn (ADT `smc-pmu` names, see [[ADT]])
	- MBSE byte 1 is the wake reason macOS reports (0 here), byte 0 = 0x30 unknown
	- Unchanged after a lid-close s2idle -> SMC doesn't see Linux sleep at all
- SMC firmware itself isn't in the IPSW (only `rt13m0` = USB-C retimer)
