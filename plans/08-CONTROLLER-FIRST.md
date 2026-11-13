# Plan 8: The character controller, first

Status: PLAN ONLY (2026-11-12). Robert, same day: "I think we should do the controller first... the actual character
controller, flying, grappling hooks, all."

Reviewed by the critic the same day: about 60 numbers checked against the game's code, nearly all confirmed; verdict pass
with fixes. This version has the fixes: every ring has pass numbers, knock-back moved to the front, the order no longer
depends on later rings, wrong numbers corrected, missing moves added.

Corrected again 2026-11-13 from executed results (job J84): the browser game's real code was run and the slice's
knock-backs were measured. Every change is listed in `R6-EXECUTED-CORRECTIONS.md`. Where the Unreal slice and the browser
game differ, both numbers are given and each is named.

This plan is the full move list and the order to build it in. Plan 5 (the obstacle course) is where each piece is proven.
Numbers are the browser game's (`D:\dev3\lsw\src`). 1 u = 19 cm. Gravity 60 u/s2.

## Where the slice stands against the full list

| Family | Moves in the browser game | In the Unreal slice |
|---|---|---|
| Ground | walk, 3 ground gears, sneak, crouch, prone, crawl, slide, combat dive, dodge, two-stage stopping, take cover, lean, play dead | walk, gears |
| Jump | jump by agility rank, late-jump grace, hard landing, multi-jump, charged super leap | basic jump, hold-to-take-off |
| Climb | ledge catch, vault, mantle, fence, ladder, wall-crawl | none |
| Flight | take-off, rise and sink, 3 air gears, power dive, flip-turn, air dodge, swoop, landing flare, crash, altitude bands, 3 flight tiers, bank-to-stop drift | take-off, rise and sink, gears, power dive, swoop, landing, crash damage |
| Grapnel | fire, reel, swing, hang, shimmy, vault up, walk up or down the line, lean out to shoot, 3 variants, pull an enemy, web zip | none |
| Glide and jets | glider, jetpack on fuel, momentum glide | none |
| Contact | body contact by shared momentum, brace before impact, tackle, grab, struggle free, carry, throw, in-flight snatch | swoop knock-back, punch step-in |
| Special | teleport and blink, burst-step, phase through walls, swim, size change | none |
| Gates | ki, needs, breath, altitude cap, wounds, load | ki, wounds |
| Camera | chase, shoulder presets, 4 aim stops, lock-on, free look | chase with aim focus |

Three places where the slice differs from the game today, to settle in ring 0:

- **Guarding slows you to x0.5 in the slice; the game uses x0.34** (`entity.js` L2681-2735).
- **Gamepad:** the slice has B as descend/swoop and RB as beam. The game has B evade and RB grab.
- **Walk speed is a difference after all (corrected 2026-11-13).** The slice walks at 32.4 u/s, which is the game's
  default 30 x 1.08 (`entity.js` L353, L2681). Run for real, the browser game's plain walk comes out at speed x 0.972 at
  60 steps a second, not x1.08: SARGE (speed 33) walks 32.08 u/s and SOL (speed 34) 33.05 u/s, so a default-30 body
  walks 29.16 u/s (`Data/cases/ground_speed_entity.json`). It also moves with the step rate (SOL: 29.38 u/s at 30 steps
  a second, 34.88 at 120). The ground gears do use the full x1.08: SOL's are 47.74, 62.42 and 80.78 u/s, which is
  34 x 1.08 x 1.3 / 1.7 / 2.2. The slice keeps 32.4 u/s at every frame rate. That is 10% faster than the browser game
  walks; it is a known difference now, not a match.

## How it is built (same method as the slice)

1. **Rules in the engine-free file, by family.** `COFCore.h` grows into a small set: ground, jump, climb, flight (exists),
   grapnel, contact, gates. Each takes "state + input + time step" and returns the new state. No Unreal types.
2. **Tests without Unreal.** Frame-by-frame tests at 30, 60, 120 and odd frame rates, like the slice's today, and compared
   against results exported from the browser game where it can be run headless.
3. **One body state table.** Which state can cancel into which (the game drops gears when grabbed, staggered, stunned,
   downed, hanging or in a vehicle; a hard landing blocks jumping; a hit interrupts a teleport). The table is data, tested,
   and drawn as a diagram for Robert.
4. **Unreal applies the result.** The hero stays on Unreal's standard character body. Things that body cannot do (grapnel
   swing, ledge hang, ladder, wall-crawl) become custom movement modes that call our rule. The swing is our own pendulum
   maths, so it is the same on every PC and at every frame rate.
5. **Proof on the course**, then a person plays it.

Engine pieces this plan uses, checked in the 5.9 install:

| Piece | Status | Use |
|---|---|---|
| Custom movement modes on the standard character body | Production | grapnel, hang, ladder, wall-crawl |
| Motion Warping | Beta, off by default | lines a vault, mantle or over-the-lip clip up with the real ledge |
| Cable component | On by default | draws the rope. Its own physics is for looks only; our maths moves the hero |
| Pose Search (motion matching), Animation Warping, Contextual Animation, Physics Control | In the install | ring-long animation work (below) |
| Mover (Epic's newer movement system) | Experimental | Not used. It ships a zipline example, but its formats can still change; our rules port to it later |

The engine has no built-in traversal, mantle or ledge system. Those rules are ours.

## Built for server and client from the start

Robert, 2026-11-12: "server client and single player like Project Zomboid." So there is one game, always run by a server.
Playing alone means your own PC runs the server quietly; playing together means joining someone's. Time passes on the
server.

What that means for the controller, decided now so nothing is rebuilt later:

| Rule | Why |
|---|---|
| The server decides where everyone is. Your PC predicts your own hero instantly and the server confirms or corrects | So controls feel immediate even with lag, and nobody can cheat their position |
| Every move runs through Unreal's built-in prediction for the standard character body. Our custom modes (boost gears, swoop, grapnel swing, hang, ladder, dives) carry their state inside each "saved move" so they can be replayed | A custom move that skips this rubber-bands under lag. The open-source PredictedMovement project (note 2) is the worked pattern for boosts, slow-downs and knock-back |
| Rules stay "state + input + time step in, new state out" | This is already how the rule file works, and it is exactly what replaying a move needs |
| Knock-back, hits, grabs and throws are decided on the server | Two players must agree who got hit and where they landed |
| Single player uses the same path | One code path to test, not two |

Every course row gains a second run under bad network conditions, using the engine's built-in lag simulator:

| Condition | Passes when |
|---|---|
| 100 ms lag | every row's numbers still within their limits; fewer than 1 visible correction per minute of play |
| 200 ms lag with 2% of messages lost | no stuck state; the hero is never out of place by more than 1 m for more than half a second; grapnel swing end points within 5 u of the no-lag run |
| Two heroes | a swoop, a tackle and a throw between two players give the same result on both screens |

Cost, stated plainly: this makes the controller roughly the hardest version of itself. It is still right to do it first,
because adding server play to a finished single-player controller means rebuilding it.

## The input rules to carry over exactly

| Input | Rule | Where |
|---|---|---|
| Gears | tap Shift 1-3 times then hold; a tap counts within 0.30 s of the last; holding the second press 0.45 s powers up | `core/movement-gears.js` L6 |
| Dodge | double-tap a direction within 0.28 s; taps under 0.04 s apart are ignored as key bounce | `core/input-intent.js` L115 |
| Dodge by state | standing: sidestep at 0.6 of evade range. Sprinting: full evade. Crouched: low roll. In the air: air dodge. Speedster: burst-step at 1.3 | `input-intent.js` L128-134 |
| Take-off | tap Space in the air, or hold Space past the top of a jump | `entity.js` L2094-2112 |
| Power dive | hold descend with a direction: +54 u/s2, sink x1.35 | `entity.js` L2189-2200 |
| Flip-turn | Z in flight: 180 degrees in 0.46 s, keeps 94% of speed, 1.1 s cooldown | `input-intent.js` L511 |
| Prone | tap Z when still; hold 0.42 s to go all the way down. Prone + Z plays dead. Q/E while aiming rolls | `engine/posture.js` L20 |
| Slide | C while sprinting: 15.8 u in 0.7 s; ends crouched if C is held | `data/reflex-graft.js` L21 |
| Combat dive | Z while moving: 2 m, 0.08 s untouchable, 1.1 s cooldown; land prone, hold C to roll to a crouch, Space to come up running | `engine/combat-dive.js` |
| Brace | Q just before an impact | key map |
| Cover and lean | C at cover takes it; Q/E lean | key map |
| Struggle | F while held | key map |
| Gamepad | A fly/jump, B evade, X strike, Y gadget, LB guard, RB grab, triggers powers, L3 gears, R3 lock; dead zone 0.24 | `core/gamepad.js` |

The PowerWorld key scheme is the one ported. The five older schemes are not.

## Build order

Every ring ends with rows on the course sheet. A ring is done when its rows pass at 30, 60, 120 and uncapped frames per
second, with scripted input and with real key events. Then Robert plays it.

### Ring 0: freeze what is good, settle what differs

| Job | Done when |
|---|---|
| Record the swoop cases. **Done (job J07):** 16 swoop cases (5 press distances x standing, guarding, hovering, plus one steep swoop) | `play_test.ps1 -Knock -Check` fails if any frozen number moves over 3% |
| Record every knock-back the slice has today. **Done (job J07):** 35 cases frozen in `Data/frozen-knockbacks.json`: the 16 swoops, 11 punches (jab, straight, finisher, haymaker, flight and dive punches) and 8 shots (ki blast, energy ball, beam) | The measured table under this one; every number is in `docs/proof/J07.md` |
| Body state table for what exists | Table and diagram exist; a test walks every listed transition |
| Settle guard slow (0.5 or 0.34), the gamepad layout, and top boost speed | One line each, with Robert's word on the pad layout |
| Top boost speed | The slice's boost III is 194.5 u/s. The browser game, run for real, is faster: SOL's top air gear holds 239 u/s under city rules and 264 u/s under open-sky rules (`Data/cases/flight_vertical_entity.json`). The 210 u/s open-sky figure is the cap with no gear held, not a limit on the gears, so "194.5 fits the 210 cap" was the wrong comparison. The game's `flight-cap.js` gives 1,322 u/s for the open world. The slice keeps 194.5 u/s inside cities (the charter's default); Plan 6 stage S0 decides the faster rungs; ring 3 uses the answer |

**What the slice's knock-backs really do** (job J07, measured at 60 frames a second, 2026-11-13). This replaces the
description that was planned from reading the code. Distances are along the ground, from where the target was hit to
where it stopped.

| Hit | On a standing target | On a guarding target | On a hovering target |
|---|---|---|---|
| Jab | 8 damage, slides 1.98 m, stays up | 1 damage, slides 0.49 m | 8 damage, drifts 1.94 m |
| Straight | 13 damage, slides 3.96 m, stays up | not run | not run |
| Combo finisher | 17 damage, launched 13.6 m, rises 1.8 m | cannot happen | not run |
| Haymaker, full charge | 34 damage, launched 31.3 m, rises 5.3 m | 11.9 damage, still launched 8.3 m | 34 damage, sent 54.8 m |
| Swoop, 5 press distances | 35.9 to 84.0 damage, moved 0.28 to 5.2 m | 4.3 to 10.1 damage, moved 2 to 4 cm; the swooper bounces at 44 to 60 u/s | 35.9 to 84.0 damage, thrown 11.1 to 17.5 m |
| Ki blast | 12 damage, 0.42 m | 5 damage, does not move | 12 damage, 0.58 m |
| Energy ball, full charge | 61.3 damage, launched 35.2 m | 25.3 damage, 7 cm | 64.4 damage, 35.8 m |
| Beam, held 1.5 s | 90 damage, moves 1 cm | not run | 51 damage, 2.3 m |

**22 of the 35 cases change by more than 3% with the frame rate** (recorded at 30, 60, 120 and uncapped). The swoop's
speed at contact depends on the frame rate, up to 15% in damage; punches thrown in flight run 27 to 32, 35 to 41 and 42
to 59 damage; the beam on a hovering target does 48 to 54; the finisher at 30 frames a second knocks down for 3.9 s
where the other rates stun for 1.7 s. **Job J82 makes every knock-back the same at every frame rate.**

**Job J83 takes the faults J07 found:** a punch thrown in flight counts its own step-in as carried speed (28 damage
from 3 m, 8 from 1.5 m); flight punches do not repeat from run to run; a blocked haymaker still lifts the guard and
throws it 8.3 m; momentum has no ceiling (a boost III jab would do about 196 damage); a swoop from a dead stop takes
its first heading from leftover velocity noise. Found by J07 and in no job yet: a close swoop on a standing target
barely moves it (28 cm), momentum changes the damage but not the knock-back, and the beam does not move a standing
target.

**The swoop differs from the browser game on purpose.** In the browser game the claw swoop ends in a hold when it goes
in from the front or the back, and throws its target only on a side cut (`Data/cases/swoop_dive_entity.json`; the
table is in Plan 5). The slice's swoop throws its target, as the table above shows. The slice's version is the one the
owner liked and it is the one kept. Only one swoop style was run in the browser game (the claw swoop, on SOL); the
levitator lunge was not.

### Ring 1: contact and knock-back

First because it is the thing Robert reacted to, and because the grapnel's enemy pull and throwing both build on it.

| Rule | Numbers | Where |
|---|---|---|
| Body contact | closing over 24 u/s while flying, bursting, sprinting or sliding; momentum shared by real body weight; damage min(28, (speed change - (14 + resilience x 3)) x 0.22); injury at most every 0.65 s | `engine/shared-impact.js` L6-33 |
| Brace | a raised guard, or Q before impact, braces | same |
| Real body weights | replace the slice's flat 80 kg; builds are hero 1.0, heavy 1.38 wide, agile 0.83, machine 1.20 | Plan 11 |
| Tackle (sprint + grab) | push against hold by body weight + half of lift, times momentum; ratio 1 succeeds, 0.55 glances; damage 4 + 6 x (closing / 26) squared, cap 16 | `tackle-rules.js`, `data/tackle.js` |
| Knock-back distance dial | one multiplier over every knock-back, default 1.0, range 1 to 3 | new, for Robert's "they go flying back really far" |

**Three kinds of knock-back** (Robert, 2026-11-12: "there are different kinds of knockbacks"). Every hit lands on one
rung of this ladder, and each rung looks and plays differently:

| Rung | What happens | What the player sees | In the slice today (measured by job J07, `Data/frozen-knockbacks.json`) |
|---|---|---|---|
| 1. Planted | Guarded. Feet stay where they are; the body absorbs it | A flinch through the arms and spine, a short skid of dust at the feet, guard meter drops, camera kick | Not yet a rung of its own. Guard cuts damage (a jab does 1 instead of 8) but the feet still move: a blocked jab slides the target 49 cm, and a blocked haymaker lifts it off its feet and throws it 8.3 m (a fault, job J83). Only a blocked swoop, ki blast or energy ball leaves the feet nearly still (0 to 7 cm) |
| 2. Driven back | You stay on your feet but are shoved backwards | Feet slide or back-step, body leans into it, recovers standing | Yes: the ground shove. A jab slides its target 198 cm and a straight 396 cm, both staying upright. A swoop pressed from 20 u moves a standing target only 28 cm |
| 3. Off your feet | Launched or knocked down | Airborne, tumble or ragdoll, land, get up | Yes: finisher 13.6 m, haymaker 31.3 m, energy ball 35.2 m; swoop 1.2 to 5.2 m on a standing target and 11.1 to 17.5 m on a hovering one. A ki blast lifts its target but carries it only 0.42 m |

**How the browser game does it: one force ladder** (run for real by job J10, `Data/cases/knockback_force_ladder.json`).
Every melee blow goes through one rule, set by the striker's strength against the target's and by the target's mass.
Jab and cross are the push class: the target slides and never leaves its feet. Finisher, power punch and haymaker are
the launch class: the target is thrown.

| Blow | Class | Base distance | Soldier on soldier (SARGE hits MERC) | Strong on weaker (SOL hits KANO) | Light on very heavy (VOLT hits TITAN) |
|---|---|---|---|---|---|
| Jab | push (slides) | 0.9 m | 0.96 m | 3.0 m | 0.14 m |
| Cross | push (slides) | 1.8 m | 1.9 m | 6.0 m | 0.29 m |
| Finisher | launch | 6 m | 6.4 m | 20.0 m | 0.95 m |
| Power punch | launch | 8 m | 8.6 m | 26.7 m | 1.3 m |
| Haymaker | launch | 14 m | 15.0 m | 46.7 m | 2.2 m |

A braced target (guarding and facing the blow) goes about 16% less far (SARGE's jab: 0.81 m), and one caught out
(airborne or staggered) about 19% further (1.14 m). The strength gap scales the distance from x0.35 to x8. These are the
three rungs as the browser game computes them, and they are what ring 1's rule ports. The slice's numbers in the table
above are a different, older rule: a slice jab shoves 1.98 m whatever the bodies weigh, because every slice body is
80 kg. Until ring 1's rule lands, the slice's frozen numbers are what the course checks. The swoop is the one knock-back
that stays the slice's own on purpose (ring 0).

Which rung a hit lands on is a rule, not an animation choice: guard up and the hit under the guard's limit gives rung 1;
a light hit or a heavy body gives rung 2; past a force threshold (by the victim's weight) gives rung 3. A guard that
breaks drops the victim to rung 2 or 3. The thresholds go in the rule file with tests, and the dial below scales rungs 2
and 3 only.

**Body contact is behind a switch.** Robert liked the look when the swoop did it, and said he wants to see bodies knocked
back. Plain contact is a different rule from the swoop, so it ships switched on for him to play, the swoop's frozen
numbers guard against it changing the swoop, and his word after playing decides if it stays.

| Course row | Passes when |
|---|---|
| Swoop freeze | the 16 frozen swoop cases within 3% (`Data/frozen-knockbacks.json`) |
| Body check, 3 weights | under 24 u/s closing: victim speed after contact under 1 u/s. Over: both speeds after contact within 5% of shared momentum |
| Brace | a braced victim moves at most 40% as far as an unbraced one of the same weight |
| Tackle | succeeds at ratio 1.0, glances at 0.55-1.0, fails under 0.55; damage within 5% |
| Knock-back table | every distance from ring 0 within 5% at dial 1.0; exactly doubled speed at dial 2.0 |
| Three rungs | the same punch on a guarding, a standing and a light dummy gives planted (feet move under 10 cm), driven back (stays upright, moves over 50 cm) and off the feet (both feet leave the ground) |
| Punching bag and force pads | a hanging bag and a row of pads show the force of every hit as a number and as swing; same hit, same number, at every frame rate |

### Ring 2: the aim camera and lock-on

Pulled forward because the grapnel and throwing both need to aim.

| Mode | Numbers | Where |
|---|---|---|
| Chase | field of view 73.74, distance 25.5 u, height 9, shoulder 0 | `data/flight-tuning.js` L7-9 |
| Shoulder presets | "backside": shoulder 6. "frontline": distance 20, field of view 68 | `data/camera-presets.js` L4, L7 |
| Aim stops | tight 8 u, tighter 4.8 u, close 9, medium 16 | `engine/aim-camera.js` L8-13 |
| Lock-on (T) | field of view 58 + 16 x speed - 6 x closeness, kept 40-76 | `engine/world.js` L2978 |
| Free look | Alt, or hold View 0.28 s | key map |
| Indoor | does not exist in the browser game. New for Unreal: pull in under a roof (Plan 2) | |

| Course row | Passes when |
|---|---|
| Aim stops | camera distance at each stop within 0.5 u; the crosshair's ray hits a 1 u target at 100 u |
| Lock-on | the locked target stays within the middle third of the screen through a full circle-strafe |
| No clipping | zero frames with the camera inside a wall across the canyon and house stations |

### Ring 3: the ground game

| Move | Numbers | Where |
|---|---|---|
| Stopping | two-stage: proportional above 17.1 u/s, then a stop in about 0.167 s | `entity.js` L205 |
| Gear response (per second) | flyer: acceleration 9, braking 5.5, turning 4. Soldier 9 / 9 / 8. Speedster 11 / 9 / 6 | `data/movement-gears.js` L14 |
| Slow-downs | crouch x0.5, sneak x0.55, crawl x0.18, guarding x0.34, mid-strike x0.5, heavy wind-up x0.4, each leg wound -9% | `entity.js` L2681-2735. The sneak x0.55 is set in `engine/input-intent-live.js` L30 and applied at `entity.js` L2689 |
| Jump | launch 25.5 / 30 / 54 / 70.5 u/s by agility rank (ranks 1-3, 4-5, 6-7, 8 and up). Run for real at 60 steps a second the apex is 5.21 / 7.25 / 23.85 / 40.83 u. The rank-3 jump is 5.00 u at 30 steps a second and 5.31 at 120, never the formula's 5.42 u; its air time is 0.85 s. Late-jump grace: the constant is 0.12 s, but a jump only still fires when pressed up to 0.10 s after the drop begins at 60 steps a second (0.108 s at 120) | `data/heights.js` L41-44. Executed: `Data/cases/jump_arc_entity.json`, `Data/cases/coyote_time_entity.json` |
| Hard landing | falling faster than 30 u/s: a knee for up to 0.26 s (half if crouched), for every body. Slam damage from a fall (`fallDamage`) applies only to soldier-type bodies (`def.archetype === 'soldier'`) or a body whose own `environment.fallDamageScale` sets it; a hero takes none from its own jump. The case body is SARGE, a soldier: its jump at agility 8 (70.5 u/s) comes down at 68.5 u/s and costs it 59 health on landing (62 at 120 steps a second); at agility 6 or 7 the 0.26 s knee and no damage; 1 to 5 neither. The same jump on a hero costs 0 health | `entity.js` L2401-2407, `weather-body.js` L10. Executed: `Data/cases/jump_arc_entity.json` (body SARGE, soldier) |
| Ledge catch | within 10.5 u of a lip the body's agility allows: 5, 9.6, 17, 34 or 51 u (`data/heights.js`). On the trunk since 2026-11-11 it is a real grab, not a snap onto the top: the hands catch the lip, a lip more than 1.3 m over the feet is hung from, then a pull-up of 1.4 s (0.8 s for a low lip). The hang lasts 0.45 s only with the stick released (0.47 s measured at 60 steps a second); with the stick held into the wall the pull-up starts early and the hang measures 0.17 s (`Data/cases/ledge_grab_entity.json`); back or crouch drops, a hit knocks the hands off, and a body falling faster than 16 u/s does not catch | `engine/ledge-grab.js` L27-41, `entity.js` L2488-2511 |
| Vault / mantle / fence | up to 6 / 14 / 14 u; motion capped at 12 u/s | `mobility-policy.js` L44 |
| Ladder | up to 80 u, on marked ladders only | `actor-traversal.js` |
| Step-up | 2.5 u x size factor | `entity.js` L2474 |
| Slide, dive, dodge, prone, play dead, cover, lean | as in the input table | |

Course rows are stations 1, 2, 3 and 11 of Plan 5, with their pass numbers (speeds within 2%, apex 5.42 u (the slice's exact jump; the browser game shows 5.21 at 60 steps) and 0.85 s
within 2%, vault at 6 u, mantle at 14 u, blocked at a 3 u step). Added rows: slide covers 15.8 u within 5%; combat dive
covers 2 m within 5% and takes no damage from a hit timed inside its 0.08 s; dodge fires on a 0.27 s double-tap and not on
a 0.29 s one.

Many of these have rules but no animation in the browser game. The purchased packs already in the project cover rolls,
dives, climbing and knockdowns.

### Ring 4: flight, finished

| Move | Numbers | Where |
|---|---|---|
| Rise / sink | 46 / 26 u/s; ground take-off pop 19 u/s | `entity.js` L212 |
| Air gears, as run | Browser game, SOL (speed 34) at 60 steps a second. City rules: 47.8 u/s with no gear, then 71.8 / 143.5 / 239.2 u/s at gears I / II / III. Open-sky rules: 52.9, then 79.3 / 158.6 / 264.4 u/s. Ki 2.6 / 6 / 12 a second. The Unreal slice: 38.9, then 58.4 / 116.7 / 194.5 u/s | `Data/cases/flight_vertical_entity.json` |
| Momentum | drag 1.8 on all axes; open-sky cap 210 u/s with no gear held; coasting banks to a stop | drag 1.8 and cap 210: `core/util.js` L112, L114 (the same drag is `entity.js` L211). The cap is applied at `entity.js` L2723. Coasting: `entity.js` L2780-2816 |
| Flight tiers | bank 0.32 / 0.5 / 0.85 rad; inertia 1.9 / 1.35 / 1.0 | `data/flight-model.js` L29-31 |
| Flip-turn, air dodge | as in the input table | |
| Landing flare | starts 3 body heights up, done at 1.15 (`engine/landing-flare.js` L18-21). A second file says 3.2 (`data/flight-model.js` L38); the engine file wins. Touchdown 9 u/s | |
| Crash | hurts from 105 u/s, full at 330; drops you out of flight above 0.6 severity | `flight-model.js` L40-48 |
| City altitude bands | levels at 8, 150, 260, 320 u; letting go of rise eases onto the nearest | `core/util.js` L62 |
| Glider | hold Space while falling: fall capped at 8 u/s, air control x1.4 | `entity.js` L2259, L2727 |
| Jetpack | 2 gears; 90 s of hover fuel; burn x1 / 1.6 / 2.4; warning at 20% | `data/flight-gear.js` L29-37 |
| Charged super leap | 0.65 s to full; up 26-88, forward 30-150 u/s; 24 ki; non-flyers | `engine/traversal-leap.js` |
| Multi-jump | 3 jumps; each 0.9 of the last x0.85; 4 ki | `data/movement-powers.js` L28 |

**Flight styles.** The flight pack we own has five styles, A to E, each with its own hover, fast flight, leans and four
dodges. The browser game already measured them (`src/data/flight-poses.js`): in A, B and C no hand leads; in D one fist
leads with the shoulders rolled 42 degrees; in E both fists lead. Robert wants each named, with a visible and a meaningful
difference. Proposed names and meanings are in research note 3; A, B and C need to be looked at in the engine before
their names are final.

Course rows: Plan 5 stations 4, 5, 7 and 8 with their numbers. Added rows: flip-turn completes in 0.46 s and keeps
94% of speed within 2%; landing touches down at 9 u/s within 1 u/s; a 330 u/s wall hit drops flight; glider fall holds
at 8 u/s; the jetpack falls at 90 s of hover within 1 s; a full super leap reaches the height its launch speed predicts
within 3%.

The 330 u/s wall hit is above the slice's top boost (194.5 u/s) and above the browser game's top air gear as run
(264 u/s), so it cannot be flown level. It needs a dive from height (Plan 5, station 7); whether the hero can reach 330
that way is not proven.

### Ring 5: the grappling hook

Built and documented in the browser game (`engine/grapnel-hook.js`, `docs/GRAPNEL_HOOK.md`). Every number below was
confirmed by the critic.

| Part | Rule |
|---|---|
| Where it sticks | Real edges found from the world's own shapes: roof edge, parapet, balcony, sill, rail, bare wall, the underside of a tree limb. Aim assist searches a 17 degree cone. Reticle: green for a ledge, amber for a wall, red for nothing |
| Range | 100 u (19 m) by default |
| Hook and reel | hook flies at 420 u/s; reel at 75 u/s, any pull kept between 0.3 and 0.85 s; over the lip in 0.34 s; crouched landing 0.6 s |
| Swing (fired in the air) | a real pendulum: gravity 60, the stick pumps 30 u/s2, drag 0.08, top speed 150 u/s, shortest rope 12 u. Jump lets go with +16 u/s and you keep your speed for 2.2 s. A swing can never end higher than it began |
| Hang | shimmy at 11 u/s; Space vaults up; C drops; firing with C held hangs instead of pulling up; one-handed weapons only |
| On the line | walk up or down the wall along the rope; lean out to shoot (`data/grapnel-variants.js`) |
| Variants | gun (range 100, hook 420, reel 75), wrist (80, 520, 95), belt (90, 360, 58, both hands free) |
| Pull an enemy | Grapple Slam drags a foe to you at 80 u/s for 12 damage (uses ring 1's contact rule on arrival) |
| Who has it | one roster hero (wrist, range 150, 1.1 s cooldown, 8 ki), creator gear, an armoury gadget with 3 charges |

In Unreal: anchors are found by sweeping the aim cone against the world and testing for a lip above the hit. The block
buildings make this exact, since every edge is on the 19 cm grid. Swing, reel and hang are three custom movement modes.
Motion Warping lines the over-the-lip clip up with the real edge.

| Course row | Passes when |
|---|---|
| Grapnel wall (ledges at 5, 17, 34, 51 u) | each ledge within range is found; reel time between 0.3 and 0.85 s; hero ends standing on top |
| Reticle | green, amber, red each shown on the right surface, 20 of 20 samples |
| Swing gap (three anchors over a 40 m pit) | crossed; top speed never over 150 u/s; no swing ends higher than it began; release adds 16 u/s within 1 |
| Hang rail | shimmy 11 u/s within 5%; vault up; drop |
| Pull the dummy | arrives at 80 u/s within 5%; takes 12 damage |
| Same at every frame rate | swing arc end points within 2 u between 30 and 120 frames per second |

How the browser game itself does against that last row (`Data/cases/grapnel_swing_entity.json`): after the same free
swing its end points at 30 and 120 steps a second are 1.35 u apart, inside the 2 u line. With the stick pumping forward
they are 2.29 u apart, outside it. So to pass on a pumped swing the Unreal pendulum has to be steadier than the browser
game's.

### Ring 6: grab, carry, throw

The full rule table is in Plan 5 (lift tiers, carry speed, throw speed and arc, damage, carrying a person, the spin that
winds a throw up to x2.1). Also here because they depend on carrying: flying with a load is blocked when the load is a
struggle; dragging a body slows you x0.5; struggling free of a hold; the in-flight snatch and jump-grab (rules to be read
out of the game before porting).

Course rows: Plan 5 station 13. Refusals, carry speed, throw speed and landing spot each within 5% of the table.

### Ring 7: the gates

Ki per gear, needs as a direct gate (Robert: the needs are the gatekeeper), breath strain, the altitude cap, load
(LOADED x0.92, OVERLOADED x0.78). Rules and pass numbers are in Plan 6 (stages S1-S2) and Plan 5 station 16.

### Ring 8: special movement

| Move | Numbers | Confirmed? |
|---|---|---|
| Teleport | target 60 u, blink 22 u; 0.35 s wind-up, 0.3 s recovery, 3 s cooldown, 10 ki; a hit interrupts; tiers 12 / 22 / 36 u | Yes |
| Burst-step | speedster dodge at 1.3 of evade range | Yes |
| Phase | hold to stay untouchable; the critic found 18 and 15 ki/s in the creator data, not the surveyed 16 | Re-read at port time |
| Wall-crawl | climb force 26 u/s2 (surveyed) | Re-read at port time |
| Web zip | straight pull, 90 u/s (surveyed) | Re-read at port time |
| Swim | surface x0.45, wade x0.62; rise 12, dive 10 u/s (surveyed) | Multipliers confirmed; speeds re-read |
| Size change | speed by the square root of size, jump by size to the 3/8 (surveyed) | Re-read at port time |

Course rows: teleport lands within 1 u of the target and is cancelled by a hit inside its 0.35 s wind-up; water rows are
Plan 5 station 17; a half-size and a double-size body each clear the jump their rule predicts within 5%.

### Alongside every ring: animation

The slice switches whole clips. Each ring's moves need bodies, built with the engine's own tools: blended locomotion
(Pose Search or a blend graph), a layered upper body for carrying and aiming, physical hit reactions and ragdoll with
get-up (Physics Control), two-person moves like tackles and grabs (Contextual Animation), and hands and feet on real
ledges and ladders (Motion Warping, Animation Warping). Measured the same way as everything else: Plan 11 stages A3-A5
give the numbers (foot slide under 5 cm, hands within 3 cm of a ledge).

## Things the browser game does not have, proposed for Robert

| Proposal | Why |
|---|---|
| Jump buffering (press jump up to 0.1 s before landing and it still fires) | Not found in the game; pairs with the late-jump grace it already has |
| A superhero landing for the player | The clip exists and police use it; the player has no such state |
| Wind in the camera at speed (field of view widening with boost) | The setting exists but is off by default |
| Wall run | Not in the game at all. Only if he wants it |

## What needs no Unreal

Ring 0's tables; every rule and test in rings 1-8; the course drawing with the new stations. Only the proof on the course,
the camera and the animation need the editor.

## Open questions for Robert

1. The four proposals above: yes or no to each.
2. Gamepad: move the slice to the game's layout (B evade, RB grab), which frees those buttons for dodge and grab?
3. After playing ring 1: does plain body contact stay on?
