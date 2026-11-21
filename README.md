# Super Mario Engine
 
**EL2003 - Computer Organization and Assembly Language - Semester Project**
 
A single-player side-scrolling platform game written in x86 16-bit Real Mode assembly (NASM) for DOSBox, using VGA Mode 13h. No libraries or game engine: we write directly to video memory and use our own keyboard and timer interrupt handlers.
 
## Team
 
| Name | Roll number | Modules built |
|---|---|---|
| Saba | 25F-0730 | 1, 2, 3, 8, 9 |
| Khadija Ashfaq | 25F-0540 | 4, 5, 10, 11, 12 |
| Mehak Fatima | 25F-0799 | 6, 7, 13, 14, 15 |
 
## Controls
 
| Key | Action |
|---|---|
| Left / Right | Move |
| Left Shift | Run |
| Space | Jump (hold for higher) |
| Up / Down, Enter | Menu |
| P | Pause / resume |
| Esc | Back to title / exit |
 
**Debug keys:** `T` sets the clock to 5 seconds, `C` sets coins to 99.
 
## Features
 
- One 60-tile level with ground, pits, bricks, "?" blocks, pipes, coins, stairs and a flag pole.
- Walk, run and jump with momentum, gravity and variable jump height (8.8 fixed point).
- Scrolling camera that follows Mario past screen x 160, never scrolls back, and stops at the end.
- Goomba and Koopa Troopa with shell kick, slide and shell kills.
- Small and Super Mario with a health system (HP 1 or 2) and a 2 second blink after being hurt.
- Coins, mushroom power-up, bricks, stomp chain, flag pole scoring and time bonus.
- HUD with score, coins, world, time, lives and health hearts.
- Title, Playing, Paused, Dying, Life-lost, Level-complete and Game-over screens.
- Flicker-free drawing through an off-screen frame buffer; restart without relaunching and a clean exit.
## Game rules
 
- Start with 3 lives, Small Mario, 300 seconds.
- Score: coin +200, stomp 100 to 8000 (chain), kick shell +400, shell kill +500, mushroom +1000, brick as Super +50, flag 100 to 5000 by height, time left x 50.
- An enemy touch shrinks Super Mario to Small; as Small it kills. Pits and the clock kill at any health.
- 100 coins give an extra life. Game over when the last life is lost.
## Game states
 
A state variable with a jump table drives the game: TITLE, PLAYING, PAUSED, DYING, LIFE LOST, WIN, LEVEL COMPLETE, GAME OVER and EXIT. P toggles pause, Esc returns to the title, and Enter moves through menus and end screens.
 
## Technical design
 
- **Memory:** single-segment `.COM` program. The frame buffer is a second 64,000-byte segment at `CS + 1000h`. The level map has an original copy and a working copy, and restart copies the original back with `REP MOVSB`.
- **Keyboard:** own INT 09h handler reading port 60h, with a 128-byte held-key table, EOI and `IRET`.
- **Timer:** own INT 08h handler at 30 Hz that chains to the BIOS about 18.2 times per second. The game loop waits on the tick counter, with no delay loops.
- **Arithmetic:** 8.8 fixed point for movement, 32-bit score with `ADD`/`ADC`, decimal digits by repeated division by 10.
- **Procedures:** parameters passed on the stack, all registers preserved.
- Both interrupt vectors and the timer rate are restored on exit.
## Deviations from the manual
 
- Super Mario is the 16 x 16 art stretched 2x vertically.
- Coin blocks give points and a coin without a pop-up animation.
- After the pole slide, Mario disappears instead of walking to a castle.
- Text uses the BIOS ROM 8 x 8 font.
- Some enemy helper procedures take the entity offset in `SI` instead of on the stack.

## Originality
 
All code in this repository was written by the team members listed above, and each member can explain any part of it.
 
