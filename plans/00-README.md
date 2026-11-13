# COF on Unreal: the plans

Written 2026-11-12. Research and planning only. Nothing in these plans has been built, and none of the first steps need
Unreal running.

**Start with [14. The roadmap](14-ROADMAP.md).** It puts everything below in one order, controller first.

| Plan | One line |
|---|---|
| [8. The character controller, first](08-CONTROLLER-FIRST.md) | Every way a character moves, the buttons and timings, and the build order: ground, flight, grappling hook, contact and throwing |
| [5. Controller obstacle course](05-CONTROLLER-COURSE.md) | 18 stations, each with a PASS/FAIL number. Freezes the swoop knock-back first; throwing rules |
| [6. Speed, needs and space](06-SPEED-NEEDS-AND-SPACE.md) | The gates on speed; ground to orbit with no cut |
| [11. Characters and animation](11-CHARACTERS-AND-ANIMATION.md) | Build on our blocky hero; a real animation setup replaces clip switching |
| [10. Powers and combat](10-POWERS-AND-COMBAT.md) | 385 powers run on about 55 behaviour types; build the types once |
| [9. Screens and the on-screen display](09-SCREENS-AND-HUD.md) | One palette (COF Tactics black), 30+ screens, in the order the game needs them |
| [2. Buildings and the house formula](02-BUILDINGS-AND-HOUSES.md) | Block buildings stay the truth. Prove the house formula by walking the hero through 1,920 houses |
| [3. Destruction](03-DESTRUCTION.md) | Same damage rules; Unreal adds falling floors, real debris, fire and light |
| [1. The world](01-WORLD.md) | 1,109 cities generated from a seed; port the generator; our own cell streamer |
| [12. People, police and vehicles](12-PEOPLE-POLICE-VEHICLES.md) | Crowds with the ten reactions, six police levels, 11 vehicles |
| [13. Saves and sound](13-SAVES-AND-SOUND.md) | Same save shape without the browser's size limit; true 3D sound |
| [4. Tools and data](04-TOOLS-AND-DATA.md) | One export script gets all game data out of the browser code |
| [7. Robert's learning path](07-LEARNING-PATH.md) | 15 engine topics in the order this game uses them |
| [Research note 1](R1-ENGINE-FACTS.md) | Eleven engine facts read from the 5.9 source that settle plan choices |
| [17. How we build it, and what to prompt](17-HOW-WE-BUILD.md) | The working loop, what we take from the "built it in a day" video and what we do differently, and short prompts for Robert |
| [16. The server and the war game](16-SERVER-AND-WAR-GAME.md) | Zomboid-style solo, friends and community servers on two layers; the browser game's war layer ported; 198 countries; time passes; seasons later |
| [15. Guns, held right](15-GUNS.md) | Six weapon classes each get their own hold; the support hand is pulled onto every gun; dive-and-shoot; people visible in cars |
| [Note 3](R3-ROBERT-DIRECTION-2026-11-12.md) | Robert's direction from 2026-11-12: purchases, three knock-backs, flight style names, the vision |
| [Research note 4](R4-WHAT-IS-POSSIBLE.md) | What is proven and what is new; destruction options; VR; water, boats, aircraft |
| [Research note 2](R2-EXISTING-CONTROLLERS.md) | Existing character controllers: what we own, what is free, what is sold. Answer: do not buy one |

## One method for all of it

The slice worked because the hero's rules live in a file with no engine in it, tested on its own, and Unreal only draws
the result. Every plan repeats that: **export real results from the browser game, port the rule to engine-free code,
test it against those results, then put it on screen.** It is why so much can be done with Unreal closed.

## Decided by Robert on 2026-11-12

- The controller comes first: "flying, grappling hooks, all."
- Research and planning only for now; no building until he says.
- The needs system is the gatekeeper for high speed (planned as a direct gate).
- The knock-back when one body flies into another is wanted (the slice has it only in the swoop; plain contact is planned).

## Still his to decide

Listed in the roadmap, in the order they start to matter.

## Checked by the critic

- Plans 1-7 (first round): pass with fixes, fixes applied. 26 of 35 numbers confirmed against the game's code; 6 wrong
  and 3 not found, all corrected or marked. Four engine facts confirmed in the 5.9 source.
- Plan 8: pass with fixes, fixes applied. About 60 numbers checked, nearly all confirmed, including every grappling
  hook number.
- Plans 9-14: pass with fixes, fixes applied. 39 of 49 claims confirmed, 5 wrong and 5 not found, all corrected or marked.
  The roadmap's five ordering mistakes are fixed.
- The corrected versions have not been re-read, and a second independent review has not happened.

## Honest limits

- Every Unreal number marked "target" is a guess to be measured, not a promise.
- Anything marked UNVERIFIED was not confirmed in the 5.9 install or by a test.
- 5.9 is a Preview build. Crowds, the vehicle system, Mover, the AI door into the editor and Chaos Cache are Experimental.
- SOVEREIGN runs one Unreal editor at a time.
