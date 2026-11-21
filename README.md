<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Super Mario Engine - README</title>
<style>
:root{--bg:#fff;--fg:#1f2328;--mut:#59636e;--bd:#d1d9e0;--hd:#1f3864;--th:#f3f6fa;--code:#f3f4f6;--hl:#fff8c5}
@media (prefers-color-scheme:dark){:root{--bg:#0d1117;--fg:#e6edf3;--mut:#9198a1;--bd:#3d444d;--hd:#9ec1ff;--th:#161b22;--code:#161b22;--hl:#3a3200}}
body{background:var(--bg);color:var(--fg);font:16px/1.6 Arial,Helvetica,sans-serif;margin:0}
main{max-width:900px;margin:0 auto;padding:24px 20px 60px}
h1{font-size:2em;color:var(--hd);border-bottom:2px solid var(--bd);padding-bottom:8px}
h2{font-size:1.4em;color:var(--hd);border-bottom:1px solid var(--bd);padding-bottom:4px;margin-top:2em}
h3{font-size:1.1em;margin-top:1.4em}
.sub{color:var(--mut)}
.note{background:var(--hl);border:1px solid var(--bd);border-radius:6px;padding:8px 12px}
.wrap{overflow-x:auto}
table{border-collapse:collapse;width:100%;margin:12px 0}
th,td{border:1px solid var(--bd);padding:6px 10px;text-align:left;vertical-align:top}
th{background:var(--th)}
pre,code{font-family:"Courier New",monospace;background:var(--code)}
code{padding:1px 5px;border-radius:4px;font-size:.92em}
pre{padding:12px;border-radius:6px;overflow-x:auto;border:1px solid var(--bd)}
pre code{padding:0;background:none}
.ph{background:var(--hl);padding:0 4px;border-radius:3px}
</style>
</head>
<body>
<main>

<h1>Super Mario Engine</h1>
<p class="sub">EL2003 - Computer Organization and Assembly Language, Semester Project</p>
<p>A single-player side-scrolling platform game written entirely in x86 16-bit Real Mode assembly (NASM), running as a <code>.COM</code> program in DOSBox. There are no libraries, engines or high-level code. Graphics go straight to VGA Mode 13h video memory, and the keyboard and timer are handled by our own interrupt routines.</p>

<h2>1. Team</h2>
<div class="wrap"><table>
<tr><th>Name</th><th>Roll number</th><th>GitHub account</th><th>Modules built</th></tr>
<tr><td><span class="ph">SABA</span></td><td><span class="ph">[25F-0730]</span></td><td><span class="ph">[1, 2, 3 ,8,9]</span></td></tr>
<tr><td><span class="ph">MEHAK FATIMA</span></td><td><span class="ph">[25F-0799]</span></td><td><span class="ph">[4, 5,10,11,12]</span></td></tr>
<tr><td><span class="ph">KHADIJA ASHFAQ</span></td><td><span class="ph">[25F-0540]</span></td><td></td><span class="ph">[6, 7 ,13,14,15]</span></td></tr>
</table></div>




<tr><td>Left arrow</td><td>4Bh</td><td>Move left</td></tr>
<tr><td>Right arrow</td><td>4Dh</td><td>Move right</td></tr>
<tr><td>Left Shift</td><td>2Ah</td><td>Run (hold)</td></tr>
<tr><td>Space</td><td>39h</td><td>Jump (hold for a higher jump)</td></tr>
<tr><td>Up / Down arrow</td><td>48h / 50h</td><td>Move menu highlight</td></tr>
<tr><td>Enter</td><td>1Ch</td><td>Select in menus, continue on end screens</td></tr>
<tr><td>P</td><td>19h</td><td>Pause / resume</td></tr>
<tr><td>Esc</td><td>01h</td><td>Back to title; exit from the title</td></tr>
</table></div>
<h3>Debug keys (listed as required by the manual)</h3>
<div class="wrap"><table>
<tr><th>Key</th><th>Effect</th></tr>
<tr><td>T</td><td>Sets the clock to 5 seconds (tests the time-out death)</td></tr>
<tr><td>C</td><td>Sets the coin counter to 99 (tests the 100-coin extra life)</td></tr>
</table></div>

<h2>5. Game summary</h2>
<ul>
<li>Start with 3 lives, Small Mario (HP 1), 300 seconds.</li>
<li>Score: coins +200, coin block +200, stomp chain 100 to 8000, kick shell +400, shell kill +500, mushroom +1000, brick (as Super) +50, flag pole 100 to 5000 by height, time left x 50 at the end.</li>
<li>Lose health: an enemy touch turns Super into Small (2 s blinking invulnerability); as Small it kills.</li>
<li>Lose a life instantly: falling into a pit or the clock reaching 0.</li>
<li>100 coins give +1 life. Winning the level does not give a life.</li>
<li>Win by touching the flag pole.</li>
</ul>

<h2>6. Game states</h2>
<p>The game is driven by a state variable and a jump table (<code>state_jt</code>) indexed by the state number.</p>
<ul>
<li><b>TITLE</b>: program start, after Game Over and after Level Complete. Up/Down move the highlight, Enter on START begins a new game, Enter on EXIT or Esc leaves to DOS.</li>
<li><b>PLAYING</b>: runs the per-frame order from Section 5.13. P goes to PAUSED, Esc to TITLE, dying to DYING, touching the flag to WIN.</li>
<li><b>PAUSED</b>: updates and the clock are frozen and "PAUSED" is drawn over the picture. P resumes, Esc returns to the title.</li>
<li><b>DYING</b>: the death animation (a hop for enemy and clock deaths, a plain fall for pits) with no collisions. It then goes to LIFE LOST if lives remain, otherwise to GAME OVER.</li>
<li><b>LIFE LOST</b>: shows the world name and lives left for about 2 seconds, then resets the level and returns to PLAYING.</li>
<li><b>WIN</b>: Mario snaps to the pole, slides down and disappears, then goes to LEVEL COMPLETE.</li>
<li><b>LEVEL COMPLETE</b>: shows score, coins, time left and the time bonus. Enter returns to the title.</li>
<li><b>GAME OVER</b>: shows the final score. Enter returns to the title.</li>
<li><b>EXIT</b>: restores the keyboard and timer vectors, returns to text mode and exits to DOS.</li>
</ul>

<h2>7. Memory choices</h2>
<ul>
<li>The program is a single-segment <code>.COM</code> file (<code>ORG 100h</code>) holding code, data and stack.</li>
<li><b>Frame buffer:</b> a second 64,000-byte segment at <code>CS + 1000h</code>, the first free segment above our 64 KB program segment. <code>AllocFrameBuffer</code> checks that it fits below <code>0A000h</code> and shows an error message if not. Everything is drawn into it and copied to video memory once per timer tick with <code>REP MOVSW</code>. It is cleared each frame with <code>REP STOSW</code>.</li>
<li><b>Level map:</b> two copies, <code>level_orig</code> (never changed after startup) and <code>level_work</code> (changed by hit blocks and taken coins). Restarting copies the original back with one <code>REP MOVSB</code>.</li>
<li><b>Tile atlas:</b> each tile is drawn once at startup (<code>BuildAtlas</code>) into a 16x16 bitmap and then copied with <code>REP MOVSB</code> each frame.</li>
<li><b>Entities:</b> one table of 10 fixed-size 16-byte records (type, state, x, y, direction, fraction, timer). The same table holds mushrooms. <code>ents_init</code> is copied over it on every level reset.</li>
<li><b>Position:</b> world x reaches 960, so Mario&rsquo;s x and y are a 16-bit integer plus an 8-bit fraction, and speeds are 8.8 fixed point. <code>ADDFIX</code> adds them with <code>ADD</code> on the fraction and <code>ADC</code> on the integer.</li>
<li><b>Score:</b> 32-bit (two words) added with <code>ADD</code>/<code>ADC</code>, clamped to 999,999.</li>
</ul>

<h2>8. Interrupts and timing</h2>
<ul>
<li><b>Keyboard (INT 09h):</b> our handler reads port 60h, ignores the E0h prefix, treats bit 7 as release, keeps a 128-byte held-key table plus a &ldquo;newly pressed&rdquo; table for menus, sends EOI and ends with <code>IRET</code>.</li>
<li><b>Timer (INT 08h):</b> the PIT is set to 30 Hz (divisor 39773). The handler counts ticks and chains to the original handler about 18.2 times per second (using an accumulator) so the DOS clock stays correct.</li>
<li>The game loop waits on the tick counter with <code>HLT</code> (no delay loops). The 300-second clock is counted from real elapsed ticks, not frames.</li>
<li>Both vectors and the PIT rate are restored on the only exit path.</li>
</ul>

<h2>9. Deviations from the manual</h2>
<ul>
<li>Super Mario uses the 16x16 Mario art stretched 2x vertically instead of a separate 16x32 bitmap.</li>
<li>Coin blocks give points and a coin directly, with no pop-up animation.</li>
<li>After the pole slide, Mario disappears instead of walking to a castle door.</li>
<li>Text is drawn with the BIOS ROM 8x8 font (<code>INT 10h</code>, <code>AX=1130h</code>), not our own font.</li>
<li>The sky colour (palette entry 9) is reprogrammed to a softer blue with <code>INT 10h</code>.</li>
<li>Several enemy helper procedures (<code>EntStep</code>, <code>EntTurnOthers</code>, <code>ShellKill</code>, <code>OverlapEnt</code>, <code>IsStomp</code>) take the entity record offset in <code>SI</code> instead of on the stack. All other procedures use stack parameters.</li>
<li>Debug keys T and C (listed above).</li>
<li><span class="ph">[Add anything else you changed]</span></li>
</ul>

<h2>10. Bonus features</h2>
<p><span class="ph">[List only the ones you completed and tested, for example walking animation, score pop-ups or a saved high score. Write &ldquo;None&rdquo; if there are none.]</span></p>

<h2>11. Known limitations</h2>
<p><span class="ph">[Write 3-5 honest limitations after testing, and how you would remove each one.]</span></p>

<h2>12. Testing</h2>
<p>Every row of the live demo checklist (manual Section 7.7):</p>
<div class="wrap"><table>
<tr><th>No.</th><th>Test</th><th>Result</th></tr>
<tr><td>1</td><td>Title menu</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>2</td><td>Walk, run, jump</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>3</td><td>Scrolling rules</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>4</td><td>Coins and coin block</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>5</td><td>Mushroom, Super form</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>6</td><td>Brick as Small and Super</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>7</td><td>Stomp chain</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>8</td><td>Hurt as Super</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>9</td><td>Hurt as Small (death)</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>10</td><td>Pit death</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>11</td><td>Clock reaches 0</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>12</td><td>Koopa shell and kill</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>13</td><td>100 coins</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>14</td><td>Flag pole and time bonus</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>15</td><td>Pause and Esc</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>16</td><td>Game over and new game</td><td><span class="ph">[pass/fail]</span></td></tr>
<tr><td>17</td><td>Clean exit</td><td><span class="ph">[pass/fail]</span></td></tr>
</table></div>

<h2>13. Originality</h2>
<p>All code in this repository was written by the team members listed above. Each member can explain any part of it.</p>

</main>
</body>
</html>
