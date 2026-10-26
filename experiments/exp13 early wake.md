exp5 + polled MBSE read / waka in `resume_noirq` (macOS wake order, before any driver)
- MBSE after real slpa = `30070100` (byte 2 = 1 new), waka reply takes 1200 us vs 300
- ANS now doesn't even boot, nvme reset -62 (ETIME), same with 1 s delay after waka
- -> SMC handshake alone isn't enough, [[S2R and lost state]]
