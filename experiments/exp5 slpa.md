slpw + slpa polled in `suspend_noirq`: resume broken
- Journal ends at suspend entry, nothing from resume hit the disk
- Came back, went black, next power on only a blinking cursor, another reboot fine
- Hold screen (smcpg `resume_hold_ms`): SMC accepts slpa (polled reply in 300 us)
- On resume NVMe is gone: `btrfs device id 1 has gone down` -> btrfs emergency shutdown -> black screen / cursor
	- ANS logs `RTKit: Initializing` but never `apcie power up done` / `pre_nand_eng enabled 1p0v`
	- nvme-apple already restarts ANS from scratch on resume on J414s, so something below it stays off
- Next: [[exp13 early wake]]
