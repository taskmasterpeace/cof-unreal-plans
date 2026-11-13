# Plan 19: Planes, ships and airports

Status: PLAN ONLY (2026-11-13). Nothing here is built in Unreal. **It starts after the controller** (ledger J90-J93):
Robert, 2026-11-13: "Once we work on the character controls, we need to start working on the ship and plane stuff,
because that's another flying thing... imagine a game where you could literally fly and ram a plane if you wanted to.
You might take damage, you might get knocked back, you might destroy both of y'all." Earlier: "we gotta be able to go
to the airport, get on a plane... ships, flying, boats, a water solution... underwater worlds."

Underwater worlds are not in this plan: they are their own feature (R4) and get their own plan. This plan leaves the
seam for them (stage F12).

Worked files beside this plan: `docs/vehicles/RAMMING.md` (the two ram laws, with cases), `docs/vehicles/ram-cases.mjs`
(makes the cases), `docs/vehicles/FAB-RESEARCH.md` (ready-made systems on Fab), `docs/vehicles/BROWSER-SURVEY.md`
(what the browser game has, file by file).

1 game unit (u) = 19 cm. Speeds in u/s; 100 u/s is 68 km/h.

## In short

- **The browser game already flies.** A fighter jet, a propeller plane and a helicopter that the player flies, AI jet
  pilots, and a walk-in 777 airliner and C-17 airlifter whose wings, engines and tail break, and which then glide down
  and crash. Airports exist (runway, terminal, check-in, security), but check-in leads to no flight. Boats do not
  exist; swimming and the seabed do.
- **Unreal gives parts, not a game.** Wing and thruster parts in its vehicle system, a floating-body part in its water
  system, all marked Experimental. Nothing for airliners, airports or ships.
- **We port the browser game's flight rules** (Robert called them "100% done and ready for tuning", 2026-09-24) into
  the engine-free rule file, as with the hero's flight, and use Unreal's physics only for wreckage. Boats get our own
  floating, engine and rudder rules on Unreal's water.
- **Ramming is decided by the server only**, by one rule for every vehicle (Robert, 2026-11-13) that uses both bodies'
  mass and the closing speed. The browser
  game's rule counts the hero's strength on some hulls but never the aircraft's mass, gives the whole hit to whichever side was
  faster, and caps the hero's hurt at 60; Robert ruled for the mass-aware rule, for every vehicle.

## What the browser game has

(Read from the export copy `_lsw-master/src` of the trunk. Full file-by-file table in `docs/vehicles/BROWSER-SURVEY.md`.)

| Thing | Built, designed or missing | In one line |
|---|---|---|
| Fighter jet, propeller plane, helicopter flown by the player | **Built** | A pure flight step with stall, afterburner (300 to 485 u/s), loops and landing gear (`engine/jet-flight.js`, `data/flight-model.js` L55-112) |
| A player's crash | **Built, thin** | The aircraft vanishes with "AIRCRAFT DISABLED" and the pilot is put out unhurt (`aircraft-piloting.js` L196-203) |
| AI fighter pilots | **Built**, one desert map only | Four jets with four flying styles (`engine/aircraft-ai.js` L418-423) |
| Shot-down aircraft | **Built** | The wreck falls burning and leaves a scorch and a blast (`engine/game.js` L2802-2852) |
| Walk-in 777 and C-17 | **Built** | Seats, doors, cockpit, lavatories, crew; every part breakable; losing an engine, a wing's lift, the tail or the pilots sends it into a glide and a crash that hurts everyone aboard (`engine/airliner/airliner.js` L270-452) |
| Airliner flights | **Built, thin** | It flies a slow circuit round its own field at 63 m (`airliner-route.js` L33), with one passenger by default (L89) |
| Background air traffic | **Designed** | A schedule (830 km/h, `data/air-travel.js` L127) that nothing in play uses |
| Airfield and terminal | **Built** | Runway, tower, hangar; check-in, security (a gun calls the police), gates, jet bridge (`world-cell/airfield-layout.js`, `data/airport.js`) |
| Ticket and trip | **Built in the old city mode only** | $60 + $0.11 a km, 900 km/h, board only at the airport, the trip skipped forward (`data/city-travel.js` L11-16). In the newer world, check-in leads to no flight |
| Airports in the city generator | **Data** | Metro cities only, a 3 x 2 cell block at the rim (`data/city-generator.js` L227-229) |
| Boats and ships | **Missing** | A car in water meets drag only (`engine/vehicle-water-contact.js` L18-24); the carrier is a speed row with no damage; harbour ships are boxes; the submarine is a room that does not move |
| Swimming, underwater view, seabed, open sea | **Built** | `actor-water-contact.js`, `underwater-camera.js`, `world-cell/seabed.js`, `world-engine/sea-ring.js` |
| Flying into a vehicle or aircraft | **Built** | Robert's contract of 2026-09-16 (`engine/shared-impact.js` L90-126): free to 30 u/s; the side that brought more speed owns the hit; on fleet vehicles the hero's strength multiplies a ram's damage (1 to 12 times, `data/force-gate.js` L18-22), but not on airliner parts; the hero's hurt is capped at 60 |
| Aircraft mass | **Missing** | No aircraft has one |
| What a crash means to the world (deaths remembered, fame, heat, news) | **Missing for aircraft** | A destroyed vehicle shows a 2.2-second news line (`vehicle-combat.js` L152); nothing else |

So the browser game has the flying and the breaking. It lacks boats, real trips after check-in, aircraft mass, and any
memory of a crash.

## What the engine gives (checked in the 5.9 install, read only, 2026-11-13)

Under `C:\Program Files\Epic Games\UE_5.9\Engine\`. Every plugin below is marked Experimental by its own plugin
file; the wing and thruster code sits in an Experimental runtime folder with no plugin file of its own; sitting on a
moving deck is standard and not Experimental.

| Part | Where | What it does | Use it? |
|---|---|---|---|
| Wing and thruster parts in the vehicle system | `Source/Runtime/Experimental/ChaosVehicles/ChaosVehiclesCore/Public/AerofoilSystem.h`, `ThrustSystem.h`; used by `Plugins/Experimental/ChaosVehiclesPlugin` (version 1.0, Experimental) | A wing surface of four kinds (fixed, wing, rudder, elevator) with area, camber, stall angle (16 degrees by default), lift and drag multipliers and an air-density ceiling; a thruster with throttle, pitch, roll and yaw. The vehicle movement part has a replicated state and a network-physics part | **No, for flight.** It is a real aerodynamics model, and we already have a tuned arcade one Robert approved. Its forces run inside the physics engine, which we do not let decide gameplay (Plan 16: one rule file, same answer on server and client). Kept as a fallback only |
| Modular vehicle | `Plugins/Experimental/ChaosModularVehicle` (version 0.1) | Vehicles built from parts (chassis, engine, wings, thrusters, wheels) that can come apart, with a replicated state | **Look at it in F5** for break-up: a plane that sheds a wing as a real part is what it is for. Version 0.1: not for anything the game depends on until it proves itself |
| Water bodies | `Plugins/Experimental/Water` (0.1), `WaterAdvanced` (0.1), `WaterExtras` (0.1, "not intended to be used directly in a shipping product"), `MeshPartitionWater` (1.0) | Ocean, lakes, rivers, shore waves, the underwater screen effect, a height query for any point | **Yes, for the look and the surface.** R4: it is a flat ocean in a bounded area; a round planet's ocean is ours |
| Floating bodies | `BuoyancyComponent.h` in the Water plugin (spherical "pontoons" sampled against the water, a maximum water force), plus `Plugins/Experimental/Buoyancy` (0.1) | Pushes physics bodies up where pontoons sit under the surface | **Only for loose things** (debris, bodies, crates). Boats and ships float by our own rule (below), so the server decides where a ship is |
| Mover and ChaosMover | `Plugins/Experimental/Mover`, `ChaosMover` | The newer movement system with network prediction | Not chosen: the controller uses CharacterMovement saved moves (Plan 8). Revisit when Mover leaves Experimental |
| Sitting in a seat, standing on a moving deck | Standard character movement (based movement) | A character carried by what it stands on | Yes: decks, aisles, ship bridges |

Nothing in the install is an airliner, an airport, a boat or a ship.

## Ready-made on Fab (read 2026-11-13; full table in `docs/vehicles/FAB-RESEARCH.md`)

12 Fab listings and Epic's own buoyancy page checked. **None lists 5.9** (5.8 is the newest). No listing showed Fab's "Created without generative AI"
badge; every listing that loaded has its "Generated with AI" field set to No.

| What | Best candidate | Listed price (Personal / Pro) | Saves real work? |
|---|---|---|---|
| Fighter jet flight | F-16 Sound System + Flight Model (same seller: F-35, F/A-18, F-15), C++ with source, claims network replication, 4.25-5.8 | $39.99 / $79.99 | **Little.** Our server decides motion, so its flight model would be replaced; the engine sound that changes with thrust and distance is the real value |
| Many planes on a server | Aircraft Battle System V2, Blueprint, its own replication; seller tested 100 AI planes and 4 players on a dedicated server | $99.99 / $139.99 | **No.** Blueprint; worth reading for how it keeps 100 planes cheap |
| AI airliner traffic | Nothing ready-made. Flying AI ($19.99, paths drawn in a level) and Flying System For Unreal ($3.99) show the usual way | - | **No.** Planes on routes are simple rules |
| Helicopters | K2 Aircraft Physics, C++ plugin, rotors only, "no Gen AI content" in its text | $159.99 / $324.99 | **No.** The browser game's helicopter is six numbers |
| Water and boats | Dynamic Real Water, C++ with source (75 classes), boats with engines and sails, "100% replicates" with the server owning the wave clock, 5.4-5.8 | $99.99 / $199.99 | **Surface boats only.** It would save a few weeks on boats and their networking, but has no pressure, no stated depth limit, no swimming, no sinking or ramming and no round planet, and stops at 5.8. Not bought; Robert ruled for Epic's free water. Full comparison: `docs/vehicles/WATER-CHOICE.md` |
| Water, free | Water Physics (C++, free, stops at 5.7); Epic's own buoyancy (free, in 5.9) | Free | Reference only |

Skipped: Fluid Flux and Driveable Ferry Boat (no multiplayer), FlightMaster (no network, 5.6 and 5.8 only), Oceanology
Legacy (syncs waves, not boats, $189.99).

## How it is built

### 1. Aircraft rules (engine-free, like the hero's flight)

The browser game's fixed-wing step (`engine/jet-flight.js`) is pure: state, controls and time in, new state out, with
no renderer. It already runs in 120 Hz sub-steps so the feel is the same at any frame rate. That is exactly the shape
of our rule file. Port it to `COFFlightCore.h` with its table (`data/flight-model.js`: jet, prop, helicopter, crash
rules) in `COFTuning.h`, each number with its source line. The helicopter, the landing gear, the crash rules and the
airliner's airworthiness rule (`airliner.js` L270-283) come with it.

Unreal then only moves the aircraft actor to where the rule says, sweeps its collision, and draws it.

### 2. Player-flown aircraft

Fighter jet, propeller plane, helicopter, with the browser game's keys (R/F throttle lever, W/S pitch, A/D roll, Q/E
rudder, Shift afterburner, Space airbrake, G gear, left mouse guns, right mouse missiles; helicopter W/S, Space up,
Ctrl down), entered at the cockpit when parked, left only when stopped on the ground (J always works). The pilot is the
hero sitting in a seat point (Plan 15's seats).

The gap to fix: in the browser game a crashed player jet simply vanishes with "AIRCRAFT DISABLED" (`aircraft-piloting.js`
L196-203). In Unreal a crash leaves a burning wreck that stays (stage F5).

A superhero can also **leave a flying aircraft** (open canopy, step out into the air and fly) and **board one in
flight** (fly to it, match speed, grab on). Both are the moving-deck rule: within 30 u/s of the deck's speed contact is
free (`shared-impact.js` L91).

### 3. Ramming a plane or a ship

The rule decides; the engine only reports the contact. The rule file gets both laws in `docs/vehicles/RAMMING.md`:

- (Robert ruled 2026-11-13: one law for every vehicle. Law B is that law; Law A is kept as recorded reference cases.)
- **Law A**, the browser game's (`shared-impact.js` L90-126, with the force gate in `data/force-gate.js` L30-46), ported exactly and frozen with cases first, as the swoop
  was. It stays the law for cars.
- **Law B**, proposed for aircraft and ships, behind a switch, needing Robert's yes: your mass in flight is your body
  plus your lift (on the game's rank ladder, where an ordinary person is rank 25: rank 30 lifts 500 kg, 40 lifts 5 t,
  50 lifts 50 t, 60 lifts 250 t) when braced, in place of Law A's strength multiplier; a hit either **breaks the part**
  and you go through it, slowed by what breaking it cost, or **bounces you** with momentum shared by the two masses. No
  cap on your hurt. Either side can die.

**Collision, ram, flying strike, flying grab** (Robert, later 2026-11-13): who gives way is decided by the browser
game's own strength contest, which already counts momentum, a raised guard and being caught out; the sub-stats decide
how well (Toughness and Grit for a collision, Guarding and Power for a ram, Striking for a flying strike, Grip and
Grappling for a flying grab), and holding costs ki or stamina. Full definitions and worked verdicts:
`docs/vehicles/FLYING-CONTACT.md`.

What ramming a plane does, by Law B's first numbers (the damage side, once the contest has decided who gives way):

| You do this | What happens |
|---|---|
| Fly braced at boost III (194.5 u/s) into a fighter jet coming at you (300 u/s) | You go through the hull and the jet breaks up. You lose 27 u/s at ordinary strength (rank 25), 18 at rank 30, and are not hurt. (Law A: everyone takes 60 and the jet flies on with 31 of its 140 points) |
| Same, but tumbling or thrown (not braced) | You still go through, but lose 134 u/s and take 130 |
| Hover in a jet's path as an ordinary person | The jet is wrecked and you die (194) |
| Tumble unbraced into an airliner's wing root | You bounce off and take 691: deadly for most bodies. The wing holds |
| A rank-30 hero through an airliner's outer wing | Through the tip panel, hurt 9. That panel carries 14% of the wing's lift, so the plane flies on; it takes the root panels, the wing box, both engines or the tail to bring it down |
| A rank-60 hero meets an airliner nose to nose | Straight through the nose without slowing, into the cockpit. A broken nose alone does not stop it flying (the browser rule counts living crew, `airliner.js` L282); if your path also hits both pilots (a slice choice: bodies in a ram's path take the hit), nobody is at the controls and it goes down. (Stopping a plane with your hands is a catch, not a ram: Plan 8's grab, not this law) |
| A hero below about rank 39 into a cargo ship's side | You bounce and take 542: steel ships are walls. From about rank 39 you hole it |
| Fly into a helicopter's rotor | The blade hits you: 8 + 0.28 x your speed (cap 34) and a fling; the rotor is damaged (Law A's rotor rule, kept) |

What the aircraft does after a hit comes from the browser game's airworthiness rule: a twin-engine plane flies on one
engine, a four-engine type on two; under 62% lift on either wing, or no tail, or nobody at the controls, it leaves its
route into a glide that ends in a crash; the crash breaks the gear and the parts it lands on and injures everyone
aboard by their fall speed (`airliner.js` L392-452).

**Break-up and burning** (new in Unreal): each aircraft model is built from its game parts (the browser game already
names them: wing panels, wing box, engines, tail, fin, nose, skin bays). A broken part detaches as a physics piece;
fire and smoke run from the broken ends; the wreck on the ground burns and stays, saved with the cell. Debris is not
saved (Plan 3's rule).

### 3b. Showing damage on every vehicle, by damage type

The game has nine damage types: physical, ballistic, energy, fire, cold, toxic, acid, magic and electric
(`src/data/damage-types.js` L2), and every hit carries one (the game's choke-point rule). Robert: "every vehicle is
going to need an efficient way of being able to show the damage from the different types." The plan:

- **Three models per vehicle are already made**: intact, damaged and destroyed, for every aircraft, mech and ground
  vehicle in the browser game's catalogue (see "Models we already have"). The model swaps at the hull's thresholds.
- **On top, one shared vehicle material** reads a small damage record per vehicle: for each zone (nose, wings, engines,
  tail, cabin; or front, sides, rear for cars) and each damage type, one number from 0 to 1. It is passed per vehicle as
  a handful of numbers (no new textures per vehicle), so a hundred damaged vehicles cost little more than one.
- **Full spec, with three levels per type, what each does to each material, and how bodies react:** `docs/vehicles/DAMAGE-LOOKS.md`.
- **What each type looks like**: physical dents and cracks; ballistic holes and chipped paint; energy scorched,
  glowing edges; fire soot, blistered paint and flames from the zone; cold frost; toxic stains and a haze; acid etched
  and running metal; magic a glow in the game's power colour; electric scorch with sparks and arcs.
- **Hits leave marks** where they land (small projected marks, oldest removed first, a fixed number per vehicle).
- **Textures in our style**: the models keep the game's faceted, low-detail look; Unreal adds better light and
  material, not more polygons. The same master material serves cars, planes, mechs and ships.
- **Saved with the cell** like any other damage (Plan 13); a burnt-out car stays burnt out.

### 3c. Ideas for a flier being shot

Robert asked for ideas from shipped games and conference talks. Mapped onto the game's own three knock-back kinds
(planted, driven back, off the feet), which in the air become **rocked, knocked off course, knocked out of the sky**.
Sources are in the shared research notes; "our idea" means no game was confirmed for it.

| # | Idea | Seen in | In Unreal | Sub-stat |
|---|---|---|---|---|
| 1 | Small flinch poses layered over the flight pose, picked by where the hit came from | Uncharted (70+ reactions this way, GDC) | An additive layer on the upper body | Toughness: smaller flinch |
| 2 | A physics layer over the animation, pushed at the bone that was hit | Uncharted 4 (GDC 2017) | Physics Control on spine and arms | |
| 3 | Big hits loosen the body for a moment, then it pulls back into the pose | Just Cause 2 (GDC) | Lower the control strength, raise it again over 0.3 s | Toughness |
| 4 | Aim knocked off separately from the body flinch, with a resistance value | Destiny 2 (stability), Titanfall 2 (per weapon) | A small camera kick per damage type | Composure: steadier aim |
| 5 | A skill that keeps the flinch you see but drops the aim penalty | Helldivers 2 ("Unflinching") | Turns off only the camera kick | A Composure signature move |
| 6 | **Knocked out of the sky**: a heavy hit drops you into a tumble, with a window to recover | Anthem | The server switches you to falling; press to recover | Toughness decides the drop, Grit the recovery |
| 7 | Missile lock warning, about 2 s to dodge | Anthem | A screen warning and the air dodge | Danger Sense: earlier warning |
| 8 | **Knocked off course** along the hit | our idea | The server adds speed along the hit | Toughness |
| 9 | Spin from hits off the centre line, which flight fights off | our idea | Roll and yaw added, damped by flight control | |
| 10 | A smoke or spark trail while badly hurt | Ace Combat | A Niagara ribbon below a health line | |
| 11 | A look per damage type: sparks for bullets, scorch for beams, a shock ring for missiles, frost for cold | common practice | One effect per damage type | |
| 12 | Camera shake and controller rumble by hit size | Titanfall 2 | Camera shake asset and force feedback | |
| 13 | The browser game's own: a flight crash over 60% severity tumbles you out of flight (`data/flight-model.js` L46) and the stun-fall Robert asked for | COF | Kept as the base rule | Grit, Toughness |

### 4. Airliners as traffic, and the airport

- **Near the player**: the browser game's 777 and C-17, with their parts, seats, doors, lavatories and crew, flying the
  airport circuit (`airliner-route.js`: rotate at 170 u/s, climb 180, circuit 150, approach 130, touchdown 125).
- **Far away**: planes on routes between real airports, drawn as cheap shapes and lights from the seed and the clock,
  so every PC sees the same plane in the same place without the server sending it. A plane is swapped for the full one
  when the player comes near, with no pop (Robert's ruling).
- **AI fighters and helicopters**: the browser game's jet pilots (`engine/aircraft-ai.js`) as enemy brains (Plan 10), and
  police and news helicopters (Plan 12's levels).
- **The airport** is the city generator's airport block (Plan 1; metro cities only, `city-generator.js` L227-229) built
  from the browser game's airfield and terminal (`world-cell/airfield-layout.js`, `data/airport.js`: regional 2 gates and
  12 flights a day, international 4 and 24), with its security arch and police rule. Buying a ticket uses the game's
  fare: $60 plus $0.11 a kilometre (`data/city-travel.js` L13-16). You must be in the airport to board (L44, L58). What
  is new: check-in leads to a real departure from a real gate.
- **Passengers**: the browser airliner carries one passenger by default. Ours fills its seats from the city's people
  (Plan 12), so a downed plane has real names on it.

### 5. The trip: the cabin as its own small level

As R4 recommends: once the doors close, the cabin is its own small level. Passengers, crew and the hero walk in it on
the moving deck; the windows show a faked outside (sky, cloud and a low ground layer at the plane's real place on the
route). The world's clock runs by the game's own rule: distance over 900 km/h (`city-travel.js` L11), through the one
time door so rent and midnights settle on the way (L1-4); the trip can be skipped, as the browser game allows.

What can happen on board: a fight, a hijack, a broken window. **If the hero leaves the plane mid-flight** (a door, a
broken window, through the skin), the cabin level hands over to the open sky: the hero appears at the plane's true
position on its route, and the plane appears as a near plane. If the plane is damaged on the trip it drops out of
"cabin mode" the same way and the airworthiness rule takes over.

A hero stays on its server (Plan 16): a flight goes to a city on the same server. The server owns the trip.

### 6. Boats and ships

Unreal has nothing for boats (R4). Ours:

- **Floating**: each hull has sample points (like the engine's pontoons); at each, the depth under our own wave surface
  gives an upward push and water drag. The wave surface is a function of the seed and the world clock, so every PC
  computes the same surface; the Water plugin draws it. The rule is in the engine-free file.
- **Engine and rudder**: thrust at the propeller, a turning force at the rudder that grows with water speed through it,
  forward and sideways drag. Our own numbers, labelled slice choice: the browser game has no boats.
- **Kinds**: speedboat, police boat, ferry, cargo ship, warship (the war layer's navy later). Each is a table row.
- **Standing on deck**, walking a ferry, a hero landing on a ship: the moving-deck rule.
- **Damage and sinking**: a ship has compartments; a hole floods its compartment, flooding lowers and tilts the hull by
  the floating rule; enough flooding and it sinks; the wreck stays on the bottom. Ramming uses Law B.
- **Today in the browser game** a car in water only meets drag (`engine/vehicle-water-contact.js`); that is the starting
  rule for a car driven into the harbour.

### 7. Consequences: the crash is remembered

Every downed aircraft and sunk ship is written into the world, by the rules the browser game already has:

- **Deaths**: every person aboard is a real body (the airliner's passengers and crew are), so deaths are counted by the
  damage door like any other; a crash on the ground hurts the people it lands on.
- **Police and the state**: a civilian hurt is +12 heat scaled by witnesses; a powered suspect seen harming people goes
  straight to the top rung the country allows (Plan 12). Downing a passenger plane puts dozens of deaths on the hero at
  once. The state's army can come for its own reasons (`src/data/state-posture.js`, `src/engine/state-army.js`).
- **Fame**: how known and how liked, per country (Plan 12).
- **The news and the war layer**: the crash becomes news, and each country's paper prints it as its press allows (Plan
  16, the newspaper). A military aircraft or ship destroyed in play is a wreck in the war world within 10 s (Plan 16,
  stage X6), and a country's relations follow the evidence of who did it.
- **The wreck stays**: on the ground or the seabed, saved with its cell (Plan 13).
- **A mass killing is a tragedy** (Robert, 2026-11-13). If the downing is seen (witnesses, cameras, the news), it is
  recorded as one event with every name on it. An investigation follows: who was seen near the plane, whose blow broke
  it (the hit credit above). The dead have funerals; the world's people who knew them (family, friends, workmates in
  the world-people simulation, Plan 12) learn of it, grieve and remember who did it. Funerals and the grief loop are
  new design, written with the death and world-people work; the pass line in F12 checks the record, the investigation
  opening and the people told.

### 8. The server decides every hit

Plan 16: server and client, the server owns every hero, and single player is a local server.

- **Flight**: the pilot's PC predicts its own aircraft with the same rule step (as the hero's custom moves use saved
  moves); the server runs the same step and corrects. AI aircraft and ships run on the server only and are sent to the
  players near them. Far traffic is not sent at all: every PC draws it from the seed and the clock.
- **A ram is decided only on the server.** Closing speeds are high: at 495 u/s, 100 ms of lag is 49.5 u (9.4 m) and
  200 ms is 18.8 m, longer than a real fighter jet (about 15 m). So the server keeps a short history (up to 200 ms) of every aircraft's and ship's
  position, rewinds the craft to where the rammer's PC saw it, sweeps the rammer's path against it, and then applies
  the rule once. The rammer's PC may show a flash and a sound at once, but the damage, the break-up and the knock-back
  happen only when the server says so.
- **One hit per pair**: the same pair cannot be hurt twice inside the browser game's window (0.45 s for vehicles,
  `shared-impact.js` L98).
- **Who did it**: the hit is credited by the same rule the browser game uses (a thrown body credits the thrower, a
  craft's hit credits its pilot; L100, L114), and that credit is what police, fame and the news read.

## Order (after J93; each stage is a game you can play)

Every Unreal stage runs its sheet at 30, 60, 120 and uncapped frames a second, by scripted and real keys, and from F3
on at 100 ms lag and at 200 ms with 2% loss (the charter's gate).

| # | Stage | Unreal? | Done when |
|---|---|---|---|
| F0 | Export the flight table, crash rules, aircraft part points, airworthiness rule, airliner routes and the ram law from the browser game; record cases on the game's real code: 6 scripted jet flights (take-off, loop, stall and recovery, burner run, landing, crash), 2 helicopter flights, 20 ram cases, 6 airliner damage cases | No | The data files and case files exist with counts in the manifest; every case replays identically twice |
| F1 | Flight rules in the engine-free file (jet, prop, helicopter, landing gear, crash, airworthiness) | No | Every F0 flight case matches within 1% in position, speed and attitude at every recorded moment, at 30, 60 and 120 steps a second and at an odd rate (47); the airworthiness cases each give the browser game's outcome; a deliberately broken number fails the tests |
| F2 | Ram rules: Law A exact, Law B behind a switch | No | Law A matches all 20 F0 ram cases within 1%; Law B reproduces every row of `RAMMING.md` exactly; the 0.45 s one-hit window holds at every step rate |
| F3 | A flyable fighter jet in the training area, server and client | Yes | Enter, take off, loop, stall and recover, land and leave, by scripted and real keys; top speed 300 and burner 485 within 2%; the wing quits at 50 u/s within 2%; every key tapped one at a time does its job; the flight sheet (each scripted manoeuvre with its numbers, written as a course sheet like Plan 8's) passes at 30, 60, 120 and uncapped, and at 100 ms lag and 200 ms with 2% loss |
| F4 | Helicopter and propeller plane; the rotor rule | Yes | Helicopter top speed 80 and climb 30 within 2%; flying into the rotor gives the rotor rule's damage within 1%; the same sheets as F3 |
| F5 | Ramming in Unreal: a target jet and a target helicopter flown by script; break-up into parts; fire; the wreck stays | Yes | A scripted ram of each kind gives the rule's damage within 1% on the server; under 200 ms lag the server's result is the same as with none and no pair is hurt twice; a broken airliner wing panel comes off as its own piece (fighters get wing, engine and tail parts too: a slice choice, the browser jet has one points pool); the wreck is still there after save and reload; all at 30, 60, 120 and uncapped |
| F6 | The 777 and C-17 outside: parts, crew, airworthiness, the disabled glide, the crash | Yes | Breaking one engine: it flies on; both: it glides down; a wing panel set under 62%: it rolls toward that side; the tail: the nose drops; both pilots down: it leaves the route; each ends as the browser game's cases do, at 30, 60, 120 and uncapped |
| F7 | Air traffic: near airliners on the circuit, far planes from the seed, AI fighters and helicopters | Yes | Two PCs show the same far plane within 1 u at the same clock; a far plane becomes a near one with zero logged pops; 4 near airliners, 8 far planes and 4 AI fighters at once keep every frame under 16.7 ms (and the most that fit is written down) |
| F8 | The airport: the sector, terminal, gate, runway; the ticket | Yes | From a street: walk in, buy a ticket for the formula's fare to the dollar, walk to the gate, board, sit; boarding outside the airport is refused with the game's message |
| F9 | The trip: cabin as its own level, faked outside, the clock, arrival; leaving in flight | Yes | Board in one city and arrive in another with the clock advanced by distance over 900 km/h within 1 s of world time; no frame over 33 ms at the switch; a hero who leaves mid-flight appears within 50 u of the plane's place on its route |
| F10 | Water and boats: a harbour on the Water plugin, our floating rule, a speedboat and a ferry | Yes | A still boat floats level within 2 degrees; the speedboat's top speed and turning circle are within 5% of its table row; two PCs agree on a boat's place within 1 u after 5 minutes; a hero standing still on the ferry's deck at full speed moves under 1 u in 60 s relative to the deck; all at 30, 60, 120 and uncapped |
| F11 | Ships: compartments, flooding, sinking, ramming ships, the wreck on the bottom | Yes | One hole floods only its compartment and the ship lists toward it by at least 2 degrees; flooding the number of compartments its table row names sinks it within 10 minutes; a ram gives Law B's result within 1%; the wreck is there after save and reload; all at 30, 60, 120 and uncapped |
| F12 | Consequences (the tragedy, investigation, the people told), and the seam for underwater | Yes | A scripted downing of an airliner over a city counts every death aboard and below, raises heat by the police rule, puts a story in the news and, for a military craft, a wreck in the war world within 10 s; a witnessed downing opens an investigation and tells the people the world records as knowing each victim; the wreck survives a restart. Swimming down to a wreck shows the engine's underwater effect (the rest is the underwater plan) |
| F13 | Shooting from a moving vehicle at a flier: the driver one-handed (pistol), a passenger two-handed (submachine gun), with Plan 15's aim layer | Yes | In a scripted chase at 60 u/s, a driver's pistol and a passenger's submachine gun each land the rule's damage on a flier at 30 m within 1%; the bike stays on its line within 1 u of the same run with nobody shooting; every fire control tapped one at a time does its job; at 30, 60, 120 and uncapped and under lag |

## Robert's rulings (2026-11-13, later the same day)

His words are quoted; each ruling replaces the default this plan first took.

| Call | Ruling | What it changes |
|---|---|---|
| One ram law or two | **One law for everything.** "We gotta have everything to be the same, like the ramming system and the weight... the mass." | Law B (mass and strength count) is the law for cars, bikes, mechs, planes and ships alike. Law A is kept only as recorded browser cases to compare against. Every vehicle needs a mass (the browser game has none for aircraft or ships; ours are slice choices to tune) |
| Can a hero die ramming a plane? | **Yes.** "The game is called Consequences of Failure for a reason... everything is measured against something... There's no cheating." | No caps on the hero's hurt from any vehicle |
| Guard up against a ramming plane; punch planes | **Yes.** "Crash into planes, punch planes, put up your guard if the plane is ramming at you." | A held guard counts as braced (the browser game's brace rule already reads the guard, `shared-impact.js` L13-20): your lift joins your mass, so a strong hero with guard up can stand off a plane that a careless one is flattened by. A punch on a plane is a strike through the same door, its force from the punch plus your mass at that moment |
| Shooting from vehicles | **Yes.** A rider steering a motorcycle fires a pistol at a flier chasing them; a passenger on the back fires a submachine gun | Plan 15's seats get firing poses: driver one-handed (pistol, machine pistol), passenger two-handed (submachine gun, rifle), with the aim layer. A stage in Plan 15's order, and a scripted chase as its pass line (below) |
| Downing a passenger plane | **The deaths are counted, and if it is seen it is a tragedy: a mass killing.** "When the investigation goes on... funerals... the people that know the characters, they gonna find out about it." | Section 7 below. Every person aboard is a named person from the world; their deaths are recorded; witnesses make it a mass killing in the police and news rules; an investigation follows (charter default had investigation parked: this ruling un-parks it for mass killings); funerals happen; the people who knew the dead learn of it. Funerals are not found in the browser game: they are new design |
| The airliner trip | **It plays in the cabin. The only skip is the time control:** in single player you can fast-forward time | No "skip trip" button. The cabin trip runs at the world's clock; single player may speed the clock up, as anywhere else |
| Boats: steal or buy | **Both.** Navies and warships are the war layer: "we don't have to simulate those with the math... off screen, and players gonna be able to be involved" | Civilian boats are stolen or bought like cars. Warships live in the war layer as numbers until players are near, then become real ships (Plan 16's seam) |
| Water | **Epic's free Water plugin.** He asked what Dynamic Real Water would save and cost: `docs/vehicles/WATER-CHOICE.md` | Not bought |
| Deep oceans and water pressure | **Needed.** "A character that can fly in space should be able to fly underwater... that got to be considered" against the sub-stats | The browser game already separates the two: space is the **vacuum** hazard, deep water the **crush** hazard (`data/environments.js` L16-34, L211-237). An energy body answers vacuum fully and crush not at all; a metal body answers some crush (L218-224). Divers have crush depths by gear: 32 m breath-hold, 66 m tank, 260 m rebreather (`data/scuba-gear.js` L34-69). So a flier who can survive space can fly underwater only as deep as their crush answer allows; past it the water crushes them, rising with depth (`engine/world-cell/dive.js` L11). Flying underwater uses water drag on flight |
| Aircraft designs | Use the game's models; change them where needed | See "Models we already have" below |

Later the same day:

| Call | Ruling | What it changes |
|---|---|---|
| A flying grab | **It is a tackle**: "flying grab would be tackling" | Four names: collision, ram, flying strike, tackle (`docs/vehicles/FLYING-CONTACT.md`) |
| Skill against rank | **Skill can beat rank**: "you've practiced it to get better than somebody who was just born with it" | Overrules the sub-stat design's "skill never beats rank". Proposed: the fitting sub-stat adds up to 2 points of contest score at 100 (a master pushes like someone four times stronger) |
| Aircraft speeds | **"Our jets are slow"** | Proposed real speeds at altitude: fighter cruise 900, dash 2,000 km/h; airliner 900 km/h; helicopters 250-300 km/h; the flight table re-tuned. Meeting a plane becomes far more dangerous (worked in FLYING-CONTACT.md) |
| Getting shot in flight | **His picks**: flinch by hit direction; aim knocked off separately, Composure steadies it; heavy hits drop you, Toughness decides the drop and Grit the recovery; **and a chance to go full ragdoll**, varied by damage type. "Whatever you can do efficiently. And lean into the stats" | Section 3c's rows 1, 4, 6, plus the ragdoll chance in `docs/vehicles/DAMAGE-LOOKS.md` |
| How damage looks | **Each type looks its own way, in levels, and reacts to our materials**: energy in the power's colours, fire with visible heat shimmer, cold frost and ice (temperature matters), toxic as poison and gas, acid and electric in levels, ballistic holes, physical from fists to bats. "This is where Unreal shows what it could do", optimised | `docs/vehicles/DAMAGE-LOOKS.md` |
| Scale | **Bigger planes; living super weapons against anything**, dinosaurs improved, a huge world in our style | Same looks and damage zones for every size; Unreal's gain goes into scale, light and material, not smoother shapes |

Still open: aircraft names (default made-up names, real kinds); airborne counting as caught (default: controlled flight
counts as standing).

## Models we already have

From the browser game's model catalogue (`public/reference-fleet/catalog.json`, read in the older working folder
`D:\dev3\lsw`, so the trunk may have more). Every one has an intact, a damaged and a destroyed version, which is what
break-up needs.

| Kind | Models |
|---|---|
| Aircraft | Strike jet, Interceptor, Two-seat escort jet, Sable stealth jet, Heavy bomber, Twenty-troop jump transport, Kestrel drone carrier, Sunbird propeller scout, Sunbird R2 biplane, Attack helicopter, Obsidian mothership (320 long, two decks, 68 capsules), Talon launch fighter |
| Airliners | The 777 and C-17 are built by code from a recipe (`airliner-recipes.js`, `airliner-model.js`), part by part |
| Mechs | Light, medium and heavy mech; the giant walkers and spider tank (built by a tool, note 210) |
| Ground | Motorcycle, ATV, armored utility, armored scout, armored cargo, fabricator, battle tank, drone tank, anti-air tank, hoverboard, hover transport |
| Defence and drones | Anti-air guns and missile batteries, radar, strategic launcher; gun, kamikaze and swarm drones |
| Sea | Aircraft carrier only. Boats and ships need new models |

They are faceted, low-detail models (about 900 to 17,600 triangles), which suits the game's blocky look. They come
into Unreal as merged meshes (Plan 12's rule), each part the game names kept separate so it can break off.

## Risks

- **Everything Unreal gives here is Experimental** (water 0.1, buoyancy 0.1, modular vehicle 0.1). Our rules carry the
  game; the engine draws.
- **Closing speeds make lag worse** than for any other fight: rewinding the craft is required, not optional.
- **Far traffic must agree on every PC**: same seed and clock rule as the city (Plan 16), whole numbers or fixed-order
  arithmetic.
- **A round planet's ocean** is not what the Water plugin does (R4). City harbours and coasts first; open ocean waits on
  Plan 6.
- **Airliners are expensive**: the browser model has hundreds of parts. Far planes are cheap shapes; only near ones are
  full.
