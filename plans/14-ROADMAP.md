# Plan 14: The roadmap, controller first

Status: PLAN ONLY (2026-11-12). This puts the plans in one order. It gives no dates: nothing has been measured that would
make a date honest. It gives order, what each step needs, and what proves it.

Re-cut the same day after the critic found five ordering mistakes in the first version. Those are fixed here: the choice
of body and the hero import now come at the start of the controller phase, the damage rules are done as paper work during
it, saves come before the destruction save test, and civilians come before crowds.

## The order, and why

Robert, 2026-11-12: "I think we should do the controller first." So the hero is finished before the world is started.
The world, buildings and destruction then arrive under a hero who already moves, fights and throws properly, and every
later system is tested by playing it with that hero.

| Phase | What | Plan stages | Ends when |
|---|---|---|---|
| **0. Groundwork** (no Unreal, bar one short run) | Export data and test results from the browser game. Freeze the swoop and every knock-back. Settle the speed ladder, guard slow and gamepad layout. Body state table. Stat formulas and the full damage door as rules | 4 (T0-T2), 8 (ring 0), 6 (S0), 10 (P0-P2), 13 (V0) | Every table exported with counts checked; one command runs all rule tests and they pass |
| **1. The controller** | First: the body decision and our hero imported with part switching. Then the rings in order: contact and knock-back, aim camera, ground, flight, grappling hook, throwing, gates, special movement. Animation alongside each ring. The display pieces and sounds the controller needs. Pause, options and rebinding | 11 (A0-A5), 8 (rings 1-8), 5, 6 (S1-S2), 9 (U1-U3), 13 (N1) | The course sheet is all PASS at 30, 60, 120 and uncapped frames per second with scripted and real key input, and Robert's play card is all thumbs up |
| **2. The roster** | Builds and sizes; the 66 roster characters; title, select and the hero creator | 11 (A1 for all 66), 9 (U4) | All 66 build with zero missing parts and each completes course stations 1-8 |
| **3. Powers** | The 13 biggest power types, then the other 16; enemy brains; power and inventory screens | 10 (P3-P7), 9 (U5) | 20 powers picked at random each hit for the exported damage within 1%; each of the 8 enemy styles defeats a standing dummy within 60 s |
| **4. A house and a wall** | Blocks, materials, the house formula, the mesher; destruction on test walls; save files for a cell | 2 (B0-B4), 3 (D0-D5), 13 (V1-V3), 11 (A6) | The 1,920-house walk passes with standard, short and large bodies; each attack leaves a hole within one block of the rule; a broken wall is the same after a restart |
| **5. A city** | The generator, the streamer, the far city, all building types; collapse, fire, levelling a house | 1 (W0-W4), 2 (B5-B7), 3 (D6-D8) | A boost III lap of a 12-cell city with no frame over 33 ms; a strength-100 hit leaves under 5% of a house's blocks |
| **6. Life** | Civilians first, then crowds, traffic, police, vehicles, the sound of the city | 11 (A7), 12 (L0-L7), 13 (N2-N4) | The crowd target Plan 12 gives after Robert's ruling; until then about 80 individually simulated people near the player with the frame under 16.7 ms, plus a measured count of how many anonymous figures and cars fit (2,000 people and 300 cars was the target until 2026-11-13); hurting a civilian brings a unit from the nearest precinct |
| **7. The country and the sky** | Highways and open land at one-eighth; weather; altitude bands | 1 (W6-W7), 6 (S3) | Miami to Orlando with no load screen and no frame over 33 ms |
| **8. Space** | Ground-to-globe blend, Earth and Moon at true size, suits, deep space | 6 (S4-S8) | One unbroken flight from a street to the Moon with no load screen |
| **9. The rest** | Map screens, devices, shops, dialogue, weapons, creatures, faces, guns and music, importing browser saves | 9 (U6-U7), 10 (P8), 11 (A8-A9), 13 (V4, N5) | Each stage's own pass line in its plan |

Two experiments are pulled forward out of their phases, because their answers could change the plans:

- **Crowds on generated roads** (Plan 12, L2). If the engine's crowd system cannot take lanes made during play, we use our
  own walkers. Cheaper to know early.
- **Ground-to-globe blend and fighting far from the centre** (Plan 6, S4-S5). Space depends on both.

## What phase 0 looks like, concretely

| # | Job | Output |
|---|---|---|
| 1 | Pick the main-branch version of the game to export from (the folder on SOVEREIGN is on a side branch) | A commit id written into the export manifest |
| 2 | Export script | Every data table as a file, counts checked |
| 3 | Results exporter | Test cases for movement and knock-back, a scripted flight, about 98 power cases, damage, houses, cities, police heat, sample saves |
| 4 | One-command test runner | Builds and runs every rule test |
| 5 | Body state table | Which state can cancel into which, as data and as a diagram |
| 6 | Speed ladder, guard slow, gamepad layout | One table, each row with its source; Robert's word on the pad |
| 7 | Knock-back freeze | The swoop's 15 cases and every other knock-back recorded, guarded by a test |
| 8 | Stat formulas and the damage door as rules | Power cases match within 1% |
| 9 | Course drawing | Every station with its size, including the grappling hook stations |

## What can run at the same time

SOVEREIGN runs one Unreal editor at a time, and Unreal cannot run while other heavy work is on. So:

- **Always possible, any seat:** exports, rule ports, tests, drawings, plans. Most of every phase starts this way.
- **One at a time, on SOVEREIGN with Unreal open:** putting a rule on screen, the course, animation, looks.

The more of each phase that is finished as rules and tests first, the shorter the time Unreal has to be open.

## Not planned yet

These exist in the browser game and no plan covers them. Leaving them out is a decision for now, not an oversight; each
needs its own plan before it is built.

| System | Where it lives in the browser game |
|---|---|
| Missions and the campaign | `src/data/missions.js`, `src/engine/missions` |
| Bases and headquarters | `src/data/base.js`, `baseroom.js` |
| Gadgets and grenades | armoury and gadget tables |
| Dogs and companions; mounts | companion and mount data |
| Robots and the transformer | `src/engine/robots` |
| How friends find and join each other, and how a server knows who you are | Usually provided by the store the game is sold on; which store is Robert's call (Plan 16) |
| Hosting official season servers | Plan 16 lists it as unplanned |
| Weather's effect on play (wind carry, storms) | weather table |
| Secret identity | identity data |
| Zombies and infection; invasion scenarios | their own modes |
| Fabrication and jobs | fabrication and employment data |
| Onboarding, accessibility, and packaging a build to hand out | docs |
| Phones and tablets (touch) | the browser game supports them; scope is Robert's call |

## All of Robert's open calls, in the order they start to matter

| When | Call | Plan |
|---|---|---|
| Phase 0 | Gamepad: move the slice to the game's layout (B evade, RB grab) | 8 |
| Phase 1, at the start | Build on our blocky hero, Manny only as a stand-in | 11 |
| Phase 1 | After playing it: does plain body contact stay on | 8 |
| Phase 1 | Jump buffering, a player superhero landing, wind in the camera at speed, wall run: yes or no to each | 8 |
| Phase 1 | Flying indoors: everywhere, big rooms only, or never under a low ceiling | 2 |
| Phase 1 | Font files for the display, or keep the stand-ins; which of the two blues | 9 |
| Phase 1 | Should sleep and rank also gate speed, on top of food and water | 6 |
| Phase 3 | The 9 known mismatches in the damage order: fix or copy first | 10 |
| Phase 3 | Ground game before or after powers | 10 |
| Phase 4 | Collapse: which of the two rulings stands | 3 |
| Phase 4 | Should damage type matter to materials | 3 |
| Phase 5 | **Crowd size. NEEDS ROBERT'S RULING** (note of 2026-11-13): the trunk went to fewer people on his word of 2026-11-07 (80 on the street, each one simulated). Default until he rules: follow the game: a small number of individually simulated people near the player (the trunk's world people), with the 2,000-figure crowd kept only as a measurement of what Unreal can afford, not a design target, until Robert rules | 12 |
| Phase 6 | Town life, rumours and ripples: in or parked. Investigation and prison: parked. Children among civilians | 12, 11 |
| Phase 7 | Travel time between cities | 1 |
| Phase 9 | Touch controls in scope. Browser saves carry over | 9, 13 |

Already answered by Robert on 2026-11-12: the controller comes first; the needs are a direct gate on speed; he wants to
see bodies knocked back on contact; **the game is server and client, with single player working the same way, like
Project Zomboid.**

## What "server and client" changes across the plans

| Plan | Change |
|---|---|
| 8 Controller | Built on the engine's movement prediction from the first ring; every course row re-run under simulated lag (details in Plan 8) |
| 1 World | The server owns the seed and the clock. Each player's PC generates the same cells from the same seed, so the world itself is never sent over the network; only changes are |
| 3 Destruction | The server decides which blocks break and sends the small list of changes. Loose debris is drawn locally and never sent |
| 13 Saves | The world save lives on the server, and so does each hero. Robert, 2026-11-12: a hero stays "on the server where it was made". A hero does not travel between servers |
| 12 People | Crowds are simulated on the server only near players; elsewhere the world advances by statistics as time passes |
| 10 Powers | Hits and damage decided on the server; effects drawn locally |
| 16 The war game | Written, and reconciled with the browser game's trunk on 2026-11-13: the trunk already has a war layer (formations, journeys, what each country knows, covert operations, a newspaper), and Unreal ports it. Its paper stages (X0-X5) port that layer against exported results and run alongside the controller. Leaders with personalities and the law and budget levers come after the port. Its "players in different cities" test is pulled forward, because it collides with Plan 1's re-centring fallback |
| Engine | A screenless community server needs the engine built from source; the installed launcher version cannot build one. Solo and hosting for friends work as installed |

## Honest limits

- No dates. The first measured thing will be phase 0's export, and the second the first ring of the controller.
- Every Unreal performance number in these plans is a target to be measured.
- 5.9 is a Preview build. Several pieces this roadmap leans on are marked Experimental by Epic (crowds, the vehicle
  system, the AI door into the editor). Each has a fallback named in its plan.
- Three critic reads were done, all by the same kind of reviewer, and all returned "pass with fixes". The fixes are
  applied, but the corrected versions have not been re-read. A second independent review has not happened.
