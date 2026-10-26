exp1 + WKTP = 2 on suspend, 0 on resume
- 2.84 h s2idle: 80% -> 75%, ~1.0 W (0.8-1.2) vs 1.9 W baseline, unclear which part helped
- MBSW = pwrbtn after resume, wake source recorded this time
- Black screen after resume: wake button press arrived after `macsmc-input` left wakeup mode -> reported as `KEY_POWER` -> niri suspended again
	- WKTP = 2 delays the button event? TBD
	- Worked around in smcpg: `KEY_POWER` from the SMC dropped for 2 s after resume
- Power number unreliable, see [[Measuring sleep power]]
