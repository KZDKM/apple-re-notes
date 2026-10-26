Final SMC power state change, sends raw messages (no key info lookup)
- State 0 (sleep):
	- Waits for SMC idle
	- Writes MBSE=slpa, or hiba if hibernating
	- Sets `_activeKeyCommand` (`this+0x130`) = 0x21, blocks all key commands until wake
- State 1 (wake):
	- Reads MBSE, returns byte 1 (wake reason)
	- Writes NTAP=1 if waking from hibernation
	- Clears `_activeKeyCommand`
