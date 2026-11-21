Super Mario Engine

EL2003 - Computer Organization and Assembly Language, Semester Project

A single-player side-scrolling platform game written entirely in x86 16-bit Real Mode assembly (NASM), running as a .COM program in DOSBox. There are no libraries, engines or high-level code. Graphics go straight to VGA Mode 13h video memory, and the keyboard and timer are handled by our own interrupt routines.


1. Team
Name	Roll number	GitHub account	Modules built
SABA	[25F-0730]		[1, 2, 3 ...]
MEHAK FATIMA	[25F-0799]		[4, 5 ...]
KHADIJA ASHFAQ	[25F-0540]		[6, 7 ...]
 Section: 3F 


Folder layout:
src/       mario.asm
docs/      report and screenshots
MARIO.COM  assembled program

4. Controls
Key	Scan code	Action
Left arrow	4Bh	Move left
Right arrow	4Dh	Move right
Left Shift	2Ah	Run (hold)
Space	39h	Jump (hold for a higher jump)
Up / Down arrow	48h / 50h	Move menu highlight
Enter	1Ch	Select in menus, continue on end screens
P	19h	Pause / resume
Esc	01h	Back to title; exit from the title
Debug keys (listed as required by the manual)
Key	Effect
T	Sets the clock to 5 seconds (test the time-out death)
C	Sets the coin counter to 99 (test the 100-coin extra life)
5. Game summary
Start with 3 lives, Small Mario (HP 1), 300 seconds.
Score: coins +200, coin block +200, stomp chain 100 to 8000, kick shell +400, shell kill +500, mushroom +1000, brick (as Super) +50, flag pole 100 to 5000 by height, time left x 50 at the end.
Lose health: an enemy touch turns Super into Small (2 s blinking invulnerability); as Small it kills.
Lose a life instantly: falling into a pit or the clock reaching 0.
100 coins give +1 life. Winning the level does not give a life.
Win by touching the flag pole.
6. Game states

The game is driven by a state variable and a jump table (state_jt) indexed by the state number.

TITLE: start of the program, after Game Over and after Level Complete. Up/Down move the highlight, Enter on START begins a new game, Enter on EXIT or Esc leaves to DOS.
PLAYING: runs the per-frame order from Section 5.13. P goes to PAUSED, Esc goes to TITLE, dying goes to DYING, touching the flag goes to WIN.
PAUSED: updates and the clock are frozen and "PAUSED" is drawn over the picture. P resumes, Esc returns to the title.
DYING: the death animation (a hop for enemy and clock deaths, a plain fall for pits) with no collisions. It then goes to LIFE LOST if lives remain, otherwise to GAME OVER.
LIFE LOST: shows the world name and lives left for about 2 seconds, then resets the level and returns to PLAYING.
WIN: Mario snaps to the pole, slides down, and disappears. It then goes to LEVEL COMPLETE.
LEVEL COMPLETE: shows score, coins, time left and the time bonus. Enter returns to the title.
GAME OVER: shows the final score. Enter returns to the title.
EXIT: restores the keyboard and timer vectors, returns to text mode and exits to DOS.
7. Memory choices
The program is a single-segment .COM file (ORG 100h) holding code, data and stack.
Frame buffer: a second 64,000-byte segment at CS + 1000h, the first free segment above our 64 KB program segment. AllocFrameBuffer checks that it fits below 0A000h and shows an error message if not. Everything is drawn into it and copied to video memory once per timer tick with REP MOVSW. It is cleared each frame with REP STOSW.
Level map: two copies, level_orig (never changed after startup) and level_work (changed by hit blocks and taken coins). Restarting copies the original back with one REP MOVSB.
Tile atlas: each tile is drawn once at startup (BuildAtlas) into a 16x16 bitmap and then copied with REP MOVSB each frame.
Entities: one table of 10 fixed-size 16-byte records (type, state, x, y, direction, fraction, timer). The same table holds mushrooms. ents_init is copied over it on every level reset.
Position: world x is up to 960, so Mario's x and y are a 16-bit integer plus an 8-bit fraction, and the speeds are 8.8 fixed point. ADDFIX adds them with ADD on the fraction and ADC on the integer.
Score: 32-bit (two words) added with ADD/ADC, clamped to 999,999.
8. Interrupts and timing
Keyboard (INT 09h): our handler reads port 60h, ignores the E0h prefix, treats bit 7 as release, keeps a 128-byte held-key table plus a "newly pressed" table for menus, sends EOI and ends with IRET.
Timer (INT 08h): the PIT is set to 30 Hz (divisor 39773). The handler counts ticks and chains to the original handler about 18.2 times per second (using an accumulator) so the DOS clock stays correct.
The game loop waits on the tick counter with HLT (no delay loops). The 300-second clock is counted from real elapsed ticks, not frames.
Both vectors and the PIT rate are restored on the only exit path.
9. Deviations from the manual
Super Mario uses the 16x16 Mario art stretched 2x vertically instead of a separate 16x32 bitmap.
Coin blocks give points and a coin directly, with no pop-up animation.
After the pole slide, Mario disappears instead of walking to a castle door.
Text is drawn with the BIOS ROM 8x8 font (INT 10h, AX=1130h), not our own font.
The sky colour (palette entry 9) is reprogrammed to a softer blue with INT 10h.
Several helper procedures for enemies (EntStep, EntTurnOthers, ShellKill, OverlapEnt, IsStomp) take the entity record offset in SI instead of on the stack. All other procedures use stack parameters.
Debug keys T and C (listed above).
[Add anything else you changed]
10. Bonus features
[List only the ones you actually completed and tested, e.g. walking animation, score pop-ups, saved high score. Leave "None" if there are none.]
11. Known limitations
[Write 3-5 honest limitations after testing, and how you would remove each one.]
12. Testing

Every row of the live demo checklist (manual Section 7.7) was tested:

No.	Test	Result
1	Title menu	[pass/fail]
2	Walk, run, jump	[pass/fail]
3	Scrolling rules	[pass/fail]
4	Coins and coin block	[pass/fail]
5	Mushroom, Super form	[pass/fail]
6	Brick as Small and Super	[pass/fail]
7	Stomp chain	[pass/fail]
8	Hurt as Super	[pass/fail]
9	Hurt as Small (death)	[pass/fail]
10	Pit death	[pass/fail]
11	Clock reaches 0	[pass/fail]
12	Koopa shell and kill	[pass/fail]
13	100 coins	[pass/fail]
14	Flag pole and time bonus	[pass/fail]
15	Pause and Esc	[pass/fail]
16	Game over and new game	[pass/fail]
17	Clean exit	[pass/fail]
13. Originality

All code in this repository was written by the team members listed above. Each member can explain any part of it.
