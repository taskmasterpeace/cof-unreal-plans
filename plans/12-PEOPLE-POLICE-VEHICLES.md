# Plan 12: People, police and vehicles

Status: PLAN ONLY (2026-11-12). Nothing here is built. Comes after the world (Plan 1) has a city on screen. The heat
rows were corrected 2026-11-13 from executed results (job J84, `R6-EXECUTED-CORRECTIONS.md`).

## People

**Today** (`src/engine/pedestrians.js`): the whole crowd is two draw orders. People walk the pavements and have ten
states: walk, flee, down, film, armed, stare, gawk, back away, call, return. Street density follows the time of day.
The fuller town life (homes, jobs, shifts, needs) exists as data that nothing uses yet. Rumours and ripples are switched
off until Robert rules.

**Trunk note, 2026-11-13. NEEDS ROBERT'S RULING.** On 2026-11-07 Robert said "I think crowds should be less", and the
trunk cut street pedestrians from 128 to 80 (`src/data/city-density.js` L22-23). The trunk also has a new system, world
people (`src/data/world-people.js`, `src/engine/world-cell/world-people.js`, `docs/WORLD-PEOPLE.md`). In two lines: a
city simulates the square root of its population as individual people (Miami 2,466, a village of 600 has 24), each
with a home, a job, a personality, needs and wants, stepped hour by hour, and the bodies near the player are those
people. Robert: "I want that kind of math more than a whole bunch of crowds that run in panic."
This plan was written for 2,000 anonymous people, which runs against that direction. The default until he rules:
Follow the game: a small number of individually simulated people near the player (the trunk's world people), with the 2,000-figure crowd kept only as a measurement of what Unreal can afford, not a design target, until Robert rules.

| Piece | State today | Plan |
|---|---|---|
| Crowd on pavements with the ten reactions | Built | Port the ten states as the crowd's brain. Far people are cheap crowd entities; within reach they become real characters |
| Homes by wealth (8 tiers), jobs, career | Built | Port the data; homes tie to the house formula (Plan 2) |
| Town life (shifts, needs per person) | Data only when surveyed. On the trunk, world people read its places, opening hours and shifts | Follows the crowd ruling in the note above |
| Dialogue wheel (9 roots) | Built | With the screens plan (Plan 9, stage U7) |
| Fame per country: how known, how liked | Built | Port as data |
| Country relations (198 x 198: 168 from Robert's own sheet, 30 derived by rule in `relations-30.js`), street organisations | Built | Port as data |
| Economy: shops, ledger, currency exchange | Built | After shops exist on screen |
| Stocks | Not found | Not planned |

**Crowds in Unreal.** The engine's crowd system is still marked Experimental in 5.9, and its lane data is normally made in
the editor, not during play. Our roads are generated during play. Research note 1 found the low-level functions needed to
build lanes during play, but it is unproven. So the first step is an experiment: one generated cell, its pavements turned
into lanes, 500 walkers on them. If it works, crowds go on the engine's system. If not, we port the browser game's own
walkers, which already work on generated streets. (2026-11-13: the 500 walkers measure what the engine can afford. They
are not a crowd target; see the trunk note above.)

## Police and law

| Piece | Numbers | State today |
|---|---|---|
| Heat from a civilian hurt | +12 is the full price; it is scaled by who saw it. With one witness each civilian hurt adds 3.96. In the spree a blast that put three people down added 36 | Built (`src/engine/police.js` L54-56). Executed: `Data/golden/police_heat_spree.json` |
| Heat from an officer | Hurt: +0.55 x the damage (30 damage adds 16.5). Killed: heat jumps to just past the next star, plus 18 for each officer killed so far: 118, then 208, then 406 in the spree. The "+40 per officer" constant is declared in the code and never used | Built. Executed: same file |
| Heat fading and stars | Heat falls 1.3 a second (gone 345 s after an unpowered spree that peaked at 406). Stars at 35, 90, 160, 240, 340, 460 | Built (`police.js` L526). Executed: same file |
| Powered and unpowered suspects | A powered suspect seen harming people goes straight to the top rung the country has: heat jumps at once to 352 in the United States and Norway, 252 in Iceland, 172 in Jamaica. An unpowered suspect never passes rung 4, whatever the heat (406 heat is five stars by heat, held at rung 4) | Built. Executed: same file |
| Six levels | beat cops, backup, SWAT, feds, military, a sanctioned superhero | Built (`police.js` L239-244) |
| Caps | each country's sheet caps the ladder; unpowered suspects top out at level 4. On the trunk the state can also send its army for its own reasons (lockdown, a presidential order, an emergency), with no suspect at all | Built (`src/data/response-policy.js`, `src/data/state-posture.js`, `src/engine/state-army.js`) |
| Stations | 1 to 5 precincts by city size, HQ and academy in big cities; units leave from the real station | Built |
| Arrest | cuffs through booking | Built (`src/engine/custody.js`) |
| Courts | a courthouse is placed in each city; trials not found | Partly |
| Investigation, prison | Designed, all marked proposed | Not built |
| Superhero law | two fields per country | Data |

Plan: heat, the six levels and the caps are rules; they go in the engine-free file with tests like everything else.
Officers are enemy brains (Plan 10) with police behaviour. Investigation and prison stay design items.

## Vehicles

| Piece | Facts |
|---|---|
| The list | 11 road vehicles: two bikes, sedan, SUV, pickup, van, stock car, coupe, bus, police cruiser, ambulance. The "26 variants" are police liveries by country. A fire engine sits outside the list |
| Handling (sedan) | top 110 u/s, acceleration 25, brake 65, grip 8.8, turn rate 1.3; weights from 220 kg to 12 t |
| Physics today | a third-party physics library: torque 450, 6,500 rpm, spring 1.8 Hz, damping 0.7 |
| Traffic | instanced, about 12 draw orders for a street; a car that is hit becomes a real physics car |
| Seats | entering, driving, hijacking at the driver's door |
| Aircraft | a jet and an airliner fly. Boats: none. Trains: designed only |
| The known cost | one standard car is about 254 draw orders, because it is built from many small pieces with their own materials |

Plan:
- **Driving** uses Unreal's vehicle system, tuned to the handling table so a sedan tops out at the same speed and stops in
  the same distance. Marked Experimental in 5.9 but widely used.
- **The 254 problem goes away by construction**: each car is imported as a few merged meshes with shared materials.
- **Traffic** is cheap crowd entities on road lanes, swapped for a real car when hit, exactly as today.
- **Thrown and rammed cars** follow the controller's rules (Plan 8, rings 1 and 6), not the vehicle system's.

**This plan owns crowds and traffic.** Plan 1's "life" stage (W5) is the same work and is done here. Until
2026-11-13 the one target was 2,000 people and 300 cars with the frame under 16.7 ms.

**Changed 2026-11-13. NEEDS ROBERT'S RULING.** The default is now to follow the game: a small number of individually simulated people near the player (the trunk's world people), with the 2,000-figure crowd kept only as a measurement of what Unreal can afford, not a design target, until Robert rules.
In numbers, until he rules: about 80 individually simulated people near the player with the frame under 16.7 ms, plus
a measured count of how many anonymous figures and cars fit in the same frame. The 500 in stage L2 is only the size of
the first experiment.

## Order

| # | Stage | Needs Unreal? | Done when |
|---|---|---|---|
| L0 | Export police rules, heat numbers, response caps, vehicle table, pedestrian states (Plan 4) | No | Counts match: 6 levels, 11 vehicles, 10 states |
| L1 | Heat and response rules in the engine-free file, with tests | No | A scripted crime spree reaches each star at the exported heat |
| L2 | Experiment: lanes built during play for one cell, 500 walkers | Yes | Walkers stay on pavements for 5 minutes; frame under 16.7 ms. Or a clear "does not work" |
| L3 | Crowd with the ten reactions: about 80 individually simulated people near the player, the trunk's world people (was "at 2,000 people" until 2026-11-13; needs Robert's ruling) | Yes | Each of the ten states is entered at least once in a scripted fight; the distances that trigger them equal the values exported from `pedestrians.js`; frame under 16.7 ms with the 80; and a measured count of how many anonymous figures fit under 16.7 ms is written down (a measurement, not a target) |
| L4 | One drivable sedan matching the handling table | Yes | Top speed and stopping distance within 5% |
| L5 | Traffic, and the hit-car swap | Yes | The game's own traffic (28 cars and 2 buses); ram one and it becomes a physics car within one frame; and a measured count of how many cars fit under 16.7 ms is written down (300 was the target until 2026-11-13; now a measurement, not a target, until Robert rules) |
| L6 | Police levels 1-3, from the real station | Yes | Hurt a civilian; a unit arrives from the nearest precinct |
| L7 | The rest of the vehicles; levels 4-6; arrest | Yes | All 11 vehicles within 5% of their table top speed and stopping distance; each of levels 4-6 arrives at its exported heat; one arrest runs cuffs to booking |

## Open questions for Robert

**First, new on 2026-11-13: how big is the crowd?** The trunk went to fewer people, each one simulated, on your word of
2026-11-07. The plan now follows that (the note under "People"); 2,000 people is only measured. Say if you want it back.

1. Town life, rumours and ripples are switched off waiting on you. In the Unreal plan or parked?
2. Investigation and prison are designed but unbuilt. Park them until the core is playable?
