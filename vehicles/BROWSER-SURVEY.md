# What the browser game has: aircraft, airports, boats, crashes

Read 2026-11-13 from the export copy of the trunk (`_lsw-master/`, commit in `Data/manifest.json`), read only. Paths are
under `_lsw-master/`. 1 u = 0.19 m. "Built" means code that runs in some mode of the game; "designed" means docs or data
nothing uses; "missing" means neither. Lines re-opened by hand are marked (checked).

## Flying: the player

| Item | State | Numbers and source |
|---|---|---|
| Fixed-wing flight step (jet and propeller), pure and renderer-free, 120 Hz sub-steps, quaternion attitude for loops and rolls | Built | `src/engine/jet-flight.js` (253 lines) (checked) |
| The flight table: hero tiers, jet, prop, helicopter, crash rules | Built | `src/data/flight-model.js`. Jet: top 300 u/s, burner x1.62 to 485, thrust 85, stall 50, rotate 68 (L60-90). Prop: top 145, stall 28 (L94-106). Helicopter: top 80, climb 30, yaw 0.95 (L109). Crash: sink rate 35, jet touchdown 85, drop 3, taxi bump 20 (L112). Hero hitting a wall in flight: harmless to 105 u/s, full at 330, at most 45% of max health, cap 260 (L40-47) (checked) |
| Piloting: enter, leave, keys, guns, missiles, gear | Built | `src/engine/aircraft-piloting.js`: R/F throttle lever, W/S pitch, A/D roll, Q/E rudder, Shift burner, Space airbrake, G gear, mouse guns and missiles (L28-49); E leaves only when stopped on the ground, J always (L23) (checked) |
| What a crash does to the player's aircraft | Built, thin | `crash()` records the impact, marks it destroyed, hides it, shows "AIRCRAFT DISABLED", ejects the pilot. No damage to the pilot found (L196-203) (checked) |
| Other aircraft rows | Built | `src/data/vehicle-envelopes.js`: jet-b 320/520, jet-c 212/329, stealth jet 219/351 (L28-31); attack helicopter 88, transport helicopter 74 (L41-42); prop scout and biplane 145 (L20-21); mothership 40 (L37) |
| Hull points | Built | `src/data/vehicle-mounts.js` L50-54: jet-a 140, jet-b 130, jet-c 150, stealth 130, helicopter 170, transport helicopter 210; fallbacks fixed-wing 150, rotor 180, ship 2,400. Parked airfield aircraft 160 (`src/engine/aircraft-combat.js` L79) (checked) |
| Aircraft mass | Missing | No mass for any aircraft anywhere in `src/` |

## Flying: AI

| Item | State | Numbers and source |
|---|---|---|
| AI fighter pilots | Built, PowerWorld desert only | `src/engine/aircraft-ai.js` `spawnAirPatrol`: 4 jets, styles professional / hound / cautious / showboat, skill 0.5 / 0.68 / 0.82 / 0.4 (L418-423); called from `src/engine/powerworld.js` L235, L360. Note 123 |
| Background air traffic on routes | Designed | `src/data/air-travel.js` (airliner 830 km/h at 10,700 m, L127) (checked) is imported only by war, economy and highway data; `src/engine/sky/sky-plane.js` has no importer. `src/data/war/units.js` L150: route pilots "next slice" |
| War-layer air units | Data | `src/data/war/units.js`: jet crew 1, armour 9 (L75); bomber 30,000 L fuel (L78); anti-air a placeholder (L147) |
| Bombing | Built | Note 191: 8 free-fall bombs from the heavy bomber, 2 from the strike jet |

## Airliners

| Item | State | Numbers and source |
|---|---|---|
| Walk-in 777 and C-17 from a recipe: seats, doors, lockable cockpit, lavatories, first class, cabin crew, rear ramp | Built | `src/engine/airliner/airliner.js` (976 lines), `src/data/airliner-recipes.js`: 777 63.7 m long, 60.9 m span (L39); C-17 53.0 m (L102) |
| Every part breakable | Built | Part points: skin 90, crown 110, belly 140, floor 120, door 150, cockpit door 260, bulkhead 200, seat 30, wing 420, engine 300, tail plane 220, fin 260, gear 160, nose 180 (`airliner-recipes.js` L171); wing box 600 (L365) (checked) |
| What the damage does (airworthiness) | Built | `airliner.js` L270-283 (checked): wing panels carry 36/30/20/14% of a wing's lift, root to tip (L52); a twin flies on one engine, a four-engine type on two; under 62% lift on either wing, no tail, or no pilot: off the route into a glide. A broken airframe will not depart |
| The disabled glide and the crash | Built | L392-452 (checked): lift from the remaining wing, the damaged side rolls down; at the ground the gear and the low wing break, everyone aboard takes fall damage plus impact; it ends as a wreck |
| Its flight | Built | A scripted circuit round its own field: rotate 170, climb 180, circuit 150, approach 130, touchdown 125 u/s, cruise height 330 u (63 m) (`airliner-route.js` L33) (checked) |
| Passengers | Built, thin | Default one passenger plus cabin crew (`airliner.js` L89) (checked) |
| Standing on a moving deck | Built | `src/engine/moving-support.js`: riders carried once per step; a blocked rider takes the vehicle impact law above 30 u/s (L20-35, L204-215) (checked) |

## Airports and trips

| Item | State | Numbers and source |
|---|---|---|
| Regional airfield in the World Cell: runway, taxiway, apron, hangar, operations, tower | Built | `src/engine/world-cell/airfield-layout.js` L65-66 (checked): runway 144 x 1,800 u (27 x 342 m); hangar 48 u tall; at most 4 parked aircraft (L16) |
| Passenger terminal: check-in, security, gates, jet bridge, baggage, evacuation | Built | `src/data/airport.js` L32-36 (checked): regional 4 desks, 2 security lanes, 2 gates, 12 flights a day; international 8, 4, 4, 24. Jet bridge 205 u (`airport-site.js` L19). A gun at the arch calls the police by the country's law (L137-145) |
| What the player does there | Built, thin | Pick up a suitcase, check in at a desk (only a line saying it is carry-on), pull the fire alarm (`airport-terminal.js` L498-528) |
| Buying a ticket and flying | Built in the old city session only | "Board passenger flight" (`src/engine/city-session/index.js` L299-301); fare $60 + $0.11/km, 900 km/h, must be at the airport (`src/data/city-travel.js` L11-16, L44, L58) (checked); the trip is skipped forward in time |
| Check-in leading to a real flight | Missing | |
| Airports placed by the city generator | Data | Metro cities only: a 3 x 2 cell block at the rim, away from highways (`src/data/city-generator.js` L227-229); sizes 1x2 / 2x2 / 2x3 (`cityplan.js` L142-144) |
| Military airstrips | Built | `world-engine/military-bases/layouts.js` L124-131 (48 u wide) |
| Customs, fuel, cargo, tower clearance, go-around | Missing | `docs/plans/AIRPORTS_AND_CARGO.md` L12-15 |
| Two passenger speeds | Conflict | 900 km/h in play (`city-travel.js` L11) and 830 km/h in the unused schedule (`air-travel.js` L127) |

## Water, boats, ships

| Item | State | Numbers and source |
|---|---|---|
| A car in water | Built | Drag only: damping 3.5 x the submerged fraction, the engine cuts out under water, no floating (`src/engine/vehicle-water-contact.js` L18-24) (checked) |
| Aircraft carrier | A motion row only | Top 16, turn 0.055 (`vehicle-envelopes.js` L69); no damage model on purpose (`vehicle-combat.js` L173) |
| Harbour ship | Decoration | A box hull (`src/engine/citytiles.js` L720-723) |
| Submarine | Built, does not move | Walk-in interior 22 m long (`world-cell/submarine-interior.js` L16-28) |
| Gunboat, ferry, patrol boat | Missing | Gunboat "future" (`war/units.js` L149) |
| Swimming, underwater camera, seabed, open sea past a coast | Built | `actor-water-contact.js`, `underwater-camera.js`, `world-cell/seabed.js`, `world-engine/sea-ring.js` |
| Navy war events | Built, crews on foot | `military-branches.js` L92-97 |
| Plans | Designed | `docs/loops/WATER-SPACE-LOOP.md` (water, submarine, carrier and mothership interiors; waits on Robert), `docs/handoffs/2026-09-21/sources/ocean-proposal.txt` |

## Ramming and crashes

| Item | State | Numbers and source |
|---|---|---|
| Body into body | Built | `src/engine/shared-impact.js` L22-45: only above 24 u/s closing (L26); impulse by both masses, factor 1.08 (L34); hurt at most once per 0.65 s, cap 28 (L11) (checked) |
| Body into vehicle or aircraft (Robert's contract 2026-09-16) | Built | L90-126 (checked): see `RAMMING.md`. Free to 30 u/s; whoever brought more closing speed owns the hit; a ram is multiplied by strength through the force gate (`data/force-gate.js` L18-36) and stopped by armour (L46); hero capped at 60 hurt (hit by a craft) or 40 (ramming); craft mass not used |
| Rotor disc | Built | L130-156 (checked): a blade, not a floor |
| Shot-down aircraft | Built | `src/engine/game.js` L2802-2852 (start checked): the wreck keeps its speed, falls, trails fire and smoke, leaves a scorch and a blast |
| A vehicle destroyed | Built | Each occupant takes 46; flash, blast sound, a 2.2 s news line "X DESTROYED"; road cars stay as burnt shells (`vehicle-combat.js` L94, L143, L152) |
| Crash consequences: fame, heat, news, police | Missing for aircraft | `airliner.js` has none; arson and terrorism are design-only rows (`law-enforcement-ladder.js` L252, L263) |
| A lasting record of incidents | Missing for vehicles | The airport keeps only its own checkpoint incidents (`airport-terminal.js` L373) |
| A doc about ramming aircraft | Missing | |

## Docs

`docs/plans/AIRPORTS_AND_CARGO.md` (audit and cargo design; partly behind the code) · `docs/AIRLINER_PASS_2026-09-22.md` ·
`docs/AIRCRAFT-FLIGHT.md` · `docs/notes/123-the-desert-air-war.md` · `docs/notes/191-bomber-run.md` ·
`docs/presidium/FLIGHT-AUDIT-v1.md` · `docs/loops/WATER-SPACE-LOOP.md` · `docs/design/WAR-LAYER-2.md` ·
`docs/FRONTLINE_MILITARY.md`, `docs/gameplay/MOTHERSHIP_HANDOFF.md`, `docs/gameplay/FLEET_EXPEDITION.md`.
