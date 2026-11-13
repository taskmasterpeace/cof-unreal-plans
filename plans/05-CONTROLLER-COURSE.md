# Plan 5: The character controller obstacle course

Status: PLAN ONLY (2026-11-12). Nothing here is built. Corrected after the critic's read (same day). Corrected again
2026-11-13 from executed results (job J84; every change is listed in `R6-EXECUTED-CORRECTIONS.md`).

## Why a course

The slice proves the controller in an empty arena with a 33-second script. That is enough for flight and punching. It is not
enough for doors, stairs, ceilings, throwing, crashing, or the house formula. The course is one map where every move the
hero has meets the thing that tests it, and every station gives a PASS or FAIL with a number.

It is run three ways, like the slice: scripted input, real key and mouse events, and by a person. It is run at 30, 60, 120
and uncapped frames per second. A station that passes at one frame rate and fails at another is a FAIL.

## The moment to protect: flying into someone

Robert, 2026-11-12, on seeing one character fly into another and knock him back: "That shit was so perfect... I've never
saw that in a game before."

**What that moment is in the slice today: the swoop.** There is no general "bodies collide" rule in the slice. The swoop
does it [`COFHero.cpp` L623-657]:

| Swoop number | Value |
|---|---|
| Strength Sk | min(1.6, speed / 58 u/s) |
| Victim is thrown | (14 + 34 x Sk) u/s along the swoop direction, plus 0.35 upward |
| Knockdown and slow motion | above Sk 0.9 |
| Victim guarding | the swooper bounces back at (18 + 26 x Sk) u/s |

**Step one was to freeze it, and that is done (job J07, 2026-11-13).** 35 knock-back cases are frozen in
`Data/frozen-knockbacks.json`. 16 of them are swoops: 5 press distances (15, 20, 26, 33 and 50 u) x a standing, a
guarding and a hovering target, plus one steep swoop from 1.6 m up. The course fails if those drift by more than 3%.
What the slice's swoop really does, measured at 60 frames a second (`docs/proof/J07.md`):

| Swoop pressed from | Speed at contact | Standing target | Guarding target | Hovering target |
|---|---|---|---|---|
| 15 u | 59 u/s | 35.9 damage, thrown 3.5 m | 4.3 damage; the swooper bounces at 44 u/s | 35.9 damage, thrown 11.1 m |
| 20 u | 69 u/s | 46.2 damage, moved 0.28 m | 5.5 damage; bounce 49 u/s | 46.2 damage, thrown 12.9 m |
| 26 u | 81.5 u/s | 61.9 damage, thrown 1.2 m | 7.4 damage; bounce 55 u/s | 61.9 damage, thrown 15.2 m |
| 33 u | 94 u/s | 80.7 damage, thrown 2.9 m | 9.7 damage; bounce 60 u/s | 80.7 damage, thrown 17.5 m |
| 50 u | 96 u/s (the cap) | 84.0 damage, thrown 5.2 m | 10.1 damage; bounce 60 u/s | 84.0 damage, thrown 17.5 m |

A swoop cannot make contact at its 34 u/s starting speed: the shortest swoop allowed (14 u) is already at about 55 u/s
when it touches. The throw is strongest on a target in the air or from a long flat approach; close to a standing target
it is mostly a stun. 22 of the 35 frozen cases, 13 of them swoops, change by more than 3% with the frame rate (the swoop
gains speed every frame and tests for contact once a frame). Job J82 fixes that; job J83 takes the other faults J07 found.

**The slice's swoop is not the browser game's swoop, and that is deliberate.** When the browser game's real code was run
(job J10, `Data/cases/swoop_dive_entity.json`), its claw swoop mostly did not throw the target:

| Browser game: the claw swoop hits | From the front or the back | From the side |
|---|---|---|
| A body the claws go into (KANO) | A hold: both bodies stopped, 50.5 to 52.5 damage, the target does not move | A pin, then released; the target does not move |
| A body the claws go into only shallowly (SOL) | The grab does not hold; 20.2 to 21 damage; the target does not move | A cut: the target is thrown at 40 to 58 u/s and travels 16 to 23.5 u |
| A body in plate (TITAN) | The claws skate off: 3.8 to 3.9 damage, the target is nudged 11 to 16 u, the swooper flies on | The same |
| A raised guard | Blocked: 0 damage, the swooper bounces off | |

So in the browser game the swoop ends in a hold from the front or the back and throws only on a side cut. The Unreal
slice's swoop throws its target from any side (the first table). **The slice's version is the one the owner liked ("That
shit was so perfect") and it is the one kept.** The browser game's hold-and-cut swoop is not ported over it. Only one
swoop style was run in the browser game (the claw swoop, on SOL); the other style, the levitator lunge, was not run, so
nothing is known from results about how it knocks back.

**Step two is a new rule, and it needs Robert's yes:** plain body contact without pressing swoop. The browser game has one
[`src/engine/shared-impact.js` L6-33], and the slice does not:

- Triggers when one side is flying, bursting, sprinting or sliding and the closing speed is over 24 u/s (4.6 m/s).
- Momentum is shared by real body mass. A raised guard braces.
- Each side takes min(28, (speed change - (14 + resilience x 3)) x 0.22). Injury at most once every 0.65 s.

Adding it means you knock people back just by flying through them, and heavier bodies move less. It must not change how the
swoop feels, which is why the swoop is frozen first. The slice's bodies all weigh 80 kg today; the rule needs real masses.

Crashing into a wall or the ground is partly in the slice (damage from 105 u/s, full at 330 u/s, 45% of health, cap 260)
[`flight-model.js` L41-46]. Dropping out of flight above 0.6 severity is not in the slice yet.

## Throwing

Not in the slice. The browser game's rules, to port into the engine-free rule file first:

| Rule | Number | Where |
|---|---|---|
| Can I lift it? | capacity / weight: under 1 refused with a reason; 1-1.6 a struggle, cannot fly; 1.6-4 heavy; 4+ easy | `carry-heft.js` L13-19 |
| Weights | lamp 0.3 t, rock 0.5, tree 1.1, car 1.9, plane 24 | note 073, `carry-heft.js` L48 |
| Carry speed | base x (1 - 0.45 / max(0.9, ratio)), kept between 0.42 and 0.93 | `game.js` L1191 |
| Throw speed (object) | 74 u/s x clamp(0.5 + 0.16 x log2(ratio), 0.5, 1.25) | `game.js` L1179 |
| Throw arc | aim direction plus 0.34 upward; gravity 62 (the hero's own jump uses 60) | `game.js` L1207-1232 |
| Object damage | base (plane 60, car 34, rock by tonnage up to 90, other 22) + STR x 3, times min(1.6, 0.75 + 0.25 x ratio) | `game.js` L1215 |
| Carrying a person | speed x max(0.45, 1 - 0.5 x mass ratio); costs ki per second; a spin winds throw speed from x1 to x2.1 over 3 s, cap 180 u/s | `person-carry.js` L50-72 |
| Person throw speed | (48, or 60 from behind) + STR x 4.6, times a weight factor 0.45-1.2 | note 060 |
| Thrown body hits a person | only above 24 u/s: the one hit takes min(30, 8 + speed x 0.22) and is launched; the thrown body takes 60% of that | `game.js` L1339-1379 |
| Into a wall | hurts above 30 u/s: min(32, (speed - 22) x 0.5); a hard enough launch breaks through instead | `entity.js` L1438-1444 |

In Unreal: a held object is attached to the hand and its physics is switched off; on release it becomes a physics body
with the launch speed from the table, and its first hit is resolved by our rule, not by the physics engine's guess. A held
person is a carried character, not a ragdoll, until released. Thrown things use the same damage door as punches. The
unbuilt parts in the browser game (swept thrown-person hits, grip strength, one hand vs two) stay out of the first pass.

## The stations

Sizes are from the browser game's own test rooms where they exist (Threat Lab, Training Hall, Flight Test Course), so the two
engines can be compared on the same course.

**The station sizes now live in `Data/course-stations.json`, which is the truth for sizes** (job J13: 29 stations, 241
plain boxes, checked by `tools/course/check_stations.py`, drawn in the docs/course folder). This table says what each station
is for and when it passes. Where a size here and the data file differ, the data file wins.

| # | Station | What is there | Passes when | Ready in the slice? |
|---|---|---|---|---|
| 1 | **Speed track** | 60 m lane, 315.8 u x 38 u | Walk and each ground gear (x1.3 / 1.7 / 2.2) reach their speed within 2%; stop distance within 5% of the rule | Yes. The slice walks 32.4 u/s. The browser game's plain walk, as run, is speed x 0.972 at 60 steps a second (`Data/cases/ground_speed_entity.json`), so the slice is 10% faster; the row is judged against the slice's own rule |
| 2 | **Jump pit** | Gaps of 1, 2, 3 m; ledges at 5, 9.6 and 17 u | Apex 5.42 u (the slice's exact jump: launch 25.5 u/s, gravity 60, `COFTuning.h` lines 22-23) and 0.85 s air time within 2%, the same at every frame rate (the browser game reaches 5.21 u at 60 steps a second only because of its stepping: 5.00 at 30, 5.31 at 120, `Data/cases/jump_arc_entity.json`); a jump pressed 0.10 s after the body leaves a ledge still fires, and one pressed 0.13 s after does not (`Data/cases/coyote_time_entity.json`) | Partly |
| 3 | **Agility** | 4 slalom posts (4 x 4 x 9 u), vault wall (40 x 3 x 4.2 u), a 6 u vault block and a 14 u mantle ledge (added 2026-11-13: the pass line asks for them and the old list had neither), 8 steps of 1.5 u to a 12 u catwalk, a drop | Through all gates (radius 14-16 u); vault at 6 u, mantle at 14 u; no frame with zero progress for over 1 s | Slalom yes; vault and mantle no |
| 4 | **Flight rings** | 16 rings on a 220 u circle at 125 +/- 35 u height, pass radius 14 u | All 16 in order at boost I, and again at boost III | Yes |
| 5 | **Canyon** | Two rows of towers 108-150 u tall, each row 42 u either side of the centre line (84 u between the rows, a 52 u clear gap), a tower every 40 u from 420 to 780 u. One 44 x 44 u pad at the canyon's mouth, 420 u before the first tower: you take off from it and come back round to land on it (as the browser game's Flight Test Course has it) | Through at boost II with zero wall touches; lands on the pad | Yes |
| 6a | **Swoop freeze** | Dummies standing, guarding, hovering; the 16 frozen swoop cases in `Data/frozen-knockbacks.json` | The recorded swoop numbers stay within 3% | Yes |
| 6b | **Body check** | Dummies of 3 weights | Under 24 u/s closing: no knock-back. Over: speeds after contact match shared momentum within 5%; guard braces | No (new rule, needs Robert) |
| 7 | **Crash wall** | A wall and a floor to hit at 80, 105, 200, 330 u/s. 330 u/s is above the slice's top boost (194.5 u/s), so that hit cannot be flown level; the drawing assumes a dive from height, and whether the hero can reach 330 that way is not proven | No damage at 80; starts at 105; full at 330; flight drops out above 0.6 severity; never inside the wall | Damage yes; drop-out no |
| 8 | **Swoop lane** | Ground targets at 3 heights | Starts between 34 and 96 u/s, connects, pulls out with at least 1 u ground clearance | Yes |
| 9 | **House walk** | 12 generated houses (3 per class) | Every room, stair and attic ladder used; no stuck spot; camera never inside a wall (Plan 2) | No |
| 10 | **Door and corridor gauge** | Doors 6 to 14 u wide in 2 u steps under 13 u lintels, corridors 6 to 12 u in 2 u steps, ceilings at 10, 16 and 24 u, and one 8 u basement stair (the stair and the door height were missing from this list until 2026-11-13) | The house formula's narrowest real openings (8 u doors, 8 u corridor, 8 u basement stair) pass on foot with at least 20 cm to spare each side | No |
| 11 | **Stairs and kerbs** | Steps of 1, 2, 2.5, 3 u; ramps of 20, 30, 45, 60 degrees | Walks up to 2.5 u without a hop; blocked at 3 u; slides off 60 degrees | Yes with one setting |
| 12 | **Indoor flight** | Warehouse (60 u ceiling), mansion foyer (47 u), normal room (16 u) | By whatever Robert rules (Plan 2, question 1) | No |
| 13 | **Lift and throw yard** | Lamp, rock, tree, car; a dummy; a target wall at 30 m | Refusals, carry speed, throw speed and landing spot each within 5% of the table above | No |
| 14 | **Punch and beam wall** | Glass, wood, brick, concrete, steel | Hole size within one block of the destruction rule (Plan 3) | No |
| 15 | **Under fire** | The shooter firing through stations 3, 4 and 9 | Staggers and recovers; no state held over 2 s; guard blocks | Yes for 3 and 4 |
| 16 | **Speed gate** | A 5 km straight, a climb to 12 km | Breath: top gear locks after 1.2 s of gasping and returns under 0.35 strain. Hunger: with Food set to 10% by a test switch, boost III lasts the predicted shorter time within 5% | No |
| 17 | **Water** | A pool 3 m deep, a river, a shore | Wade x0.62, swim x0.45 within 5%; dive and surface; fly out | No |

So the course can start with stations 1, 4, 5, 6a, 8 and 11, and parts of 2, 3, 7 and 15. The rest wait for their rules.

Two things station 16 needs that do not exist yet: a test switch to set any need to a value (starving for real takes about
90 real minutes), and a hero fast enough to meet the 1,678 mph cap, which today's game admits "binds nobody".

## What it measures every run

One row per station: PASS or FAIL, the number measured, the number wanted, the frame rate, the input mode. Plus the
always-on checks the slice already has (no broken numbers, no teleports, never through the floor, never stuck for more
than 2 seconds). Results go to one sheet per build, so two builds can be compared line by line.

## What a person checks that a script cannot

A short card for whoever plays it (Robert first): does take-off feel instant; does the camera fight you in the canyon; is
the swoop as good as it was; does a door feel wide or tight; is landing where you meant. One line per station, thumbs up
or down, and one sentence if down.

## Controller work this plan implies

| Move | In the slice today | Needed for |
|---|---|---|
| Fly, boost gears, swoop, land, crash damage | Yes | - |
| Ground gears (x1.3 / 1.7 / 2.2) | Yes | station 1 |
| Punch combo, haymaker, blasts, ball, beam, guard | Yes | - |
| Jump with late-jump grace | Partly | station 2 |
| Crash drops you out of flight | No | station 7 |
| Crouch, slide, dive | No | station 3 |
| Vault, mantle, ledge hang, ladder | No | stations 3, 9 |
| Indoor camera | No | stations 9, 10, 12 |
| Body contact by shared momentum, real body weights | No | station 6b |
| Grab, carry, throw | No | station 13 |
| Swim | No | station 17 |
| Breath and needs gates on speed | Ki only | station 16 |
| Autopilot that can find its way through a house | No | station 9 |

Each new move goes into the engine-free rule file first, with tests, and only then into Unreal. That is how the slice got
to the same result at every frame rate, and it is why the first half of this work needs no Unreal running.

## Order

The order the moves are built in is Plan 8's rings (contact and knock-back first, then the aim camera, ground, flight,
grappling hook, throwing). Where the stages below name a different order, Plan 8 wins. The slice's walk speed is settled
at 32.4 u/s (the game's 30 x 1.08). The browser game's own plain walk, as run, is lower: speed x 0.972 at 60 steps a
second (Plan 8). Plan 8 adds stations for the grappling hook, the aim camera and teleport.

## Stages

| # | Stage | Needs Unreal? | Done when |
|---|---|---|---|
| C0 | Freeze the swoop. Done by job J07: 35 knock-back cases, 16 of them swoops, in `Data/frozen-knockbacks.json` | One short run | Table saved; `play_test.ps1 -Knock -Check` fails if any number moves over 3% |
| C1 | Keep the slice's walk speed of 32.4 u/s (the browser game as run is speed x 0.972) and add to the rule file, with tests: jump, crouch, slide, vault, mantle, ladder, crash drop-out, carry, throw | No | Every number in the tables above has a passing test |
| C2 | Body-contact rule and body weights in the rule file (after Robert's yes) | No | Tests match `shared-impact.js` for 10 cases |
| C3 | Course drawing: top-down, every station's size in units and metres. Done by job J13 (`Data/course-stations.json`, with the drawings in the docs/course folder) | No | Every station in the table has a box on the drawing with its size |
| C4 | Build the ready stations as plain boxes; grow the autopilot into a station runner | Yes | First PASS/FAIL sheet at 30/60/120/uncapped |
| C5 | Robert plays it; his card decides what gets tuned | Yes | Card filled in |
| C6 | Remaining stations as their rules and plans land; autopilot house routing for station 9 | Yes | Sheet has all 18 rows of this plan's table; with Plan 8's 27 added rows the whole course is 45 rows on 29 stations |
