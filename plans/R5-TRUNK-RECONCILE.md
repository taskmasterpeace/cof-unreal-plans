# R5: The plans checked against the browser game's trunk

Written 2026-11-13 for ledger job J04. The plans (01 to 17, R1 to R4) were written from a checkout of the browser game
that was 515 changes behind its trunk. This note says what was checked against the trunk, what was fixed, what is new on
the trunk, and what was not checked.

- Trunk: `claude/uber7-2026-10-03`, commit `c733c398`, exported to `_lsw-master` (source and docs only).
- What changed between the surveyed checkout and the trunk: `docs/proof/J04-trunk-diff-names.txt` (438 files: 135
  added, 295 changed, 8 deleted).
- The check: `docs/proof/J04-check-citations.py`; its output after the fixes: `docs/proof/J04-citations.txt`.

A critic read the first version of this note and failed it: the check then only proved that cited files exist. The
check was rewritten on 2026-11-13 to test line numbers and quoted numbers; section 1 says what it does and does not
prove. The rewritten note has not been read by a critic. Line numbers were re-opened on the trunk export for every
fix listed below.

## 1. What the check does, and its totals

The script reads every plan except the ledger and this note. A **file:line citation** is a backticked path of the
browser game with a line reference straight after it (`entity.js` L983, L12-40, "line 123", `x.js:123`), or
"note 206 L22" (the numbered note under `docs/notes`). For each one the script:

1. finds the file on the trunk export;
2. checks every cited line is inside the file;
3. collects the numbers the plan quotes on that line or table row (integers of two or more digits, and all decimals;
   not the line numbers, dates, years, "note 073", "#1457" or "Plan 16"; "55%" is also tried as 0.55) and tests
   whether at least one of them is written in the cited lines, widened by 3 lines either side.

Final totals (the saved output, `docs/proof/J04-citations.txt`; exit code 0):

| Class | Count | Meaning |
|---|---|---|
| MATCH | 79 | In range, and a quoted number is at the cited lines |
| IN-RANGE-ONLY | 56 | In range; the plan line quotes no number to test |
| NO-MATCH | 10 | In range, but none of the quoted numbers is there. All ten are explained below |
| OUT-OF-RANGE | 0 | A cited line is past the end of the file |
| FILE-MISSING | 0 | The cited file is not on the trunk |
| **Total checked** | **145** | |

Beside those: 284 backticked game paths in all (174 distinct); 276 resolve to a file, 6 to a folder, none is missing.
Two are under `public/` (Plan 11: the hero model folder and its motion banks). The export has no `public` folder, so
they are listed and ignored, and never fail the run. 22 line references follow engine or project paths (`.h`, `.cpp`)
and are not this script's job. No line reference is left without a path in front of it.

The first run of the new script gave 62 MATCH, 38 IN-RANGE-ONLY, 13 NO-MATCH, 0 OUT-OF-RANGE and one missing file, and
left 26 line references unchecked because they had no path in front of them.

**What this proves.** MATCH: the file exists, the lines exist, and at least one number the plan quotes on that line is
written within 3 lines of where the plan points. IN-RANGE-ONLY: only that the file and lines exist.

**What it does not prove.** A number appearing nearby is evidence, not proof of meaning. Only one quoted number has to
be found, so other numbers on the same plan line may be wrong. Short numbers (12, 20, 30) can match by chance: Plan 8's
momentum row passed before its fix because an unrelated 1.8 sits in the cited range. Three of the critic's finds were
one line off and inside the 3-line margin, so the script did not flag them; they were fixed by hand. Where a plan
sentence wraps, only the line holding the citation is read for numbers. The 56 IN-RANGE-ONLY citations (40 of them in
Plan 16, mostly prose) were not content-checked by the script.

**The ten NO-MATCH, explained.** In each case the number on the plan line is not a literal at the cited lines:

| Plan line | Citation | Quoted | Why it does not match |
|---|---|---|---|
| 02:14 | `city-recipe-catalog.js` L41-53 | 22 | A count: the seven seed lists on L42-48 hold 22 seeds (counted in Node; 22 x 4 facings = 88) |
| 02:73 | `citylook/camera.js` L70 | 200 | 200 cm is this plan's own proposal in the next sentence. L70 is the ceiling clamp |
| 03:17 | `construct-kit.js` L417 | 1.5 | The cited fact is "x6" (one digit, not tested; read at L417: damage x power x 6). 1.5 is the hole-size column |
| 03:18 | `force-feel.js` L126 | 30 | L126 is the through-the-wall function. The 30 u/s is at the other citation on that row (`entity.js` L1438-1444, MATCH) |
| 06:12 | `entity.js` L1420, L1922 | 2.6, 12, 15%, 0.7, 0.6 | These lines are the rules (spending ki; needs cutting ki regen). The numbers are in the two data files on the same row, both MATCH |
| 08:212 | `entity.js` L2723 | 1.8, 210 | L2723 applies the cap by name (`PW_AIR.top`). The literal 210 is `core/util.js` L114 (MATCH) |
| 10:8 | `abilities.js` L407-1576 | 29 | A count of handlers the roster uses. Derived, and not recounted (section 5) |
| 16:77 | `power-elite.js` L3, L69 | 11 | "The lore names 11" is from the lore survey, not this file. L69 lists the 3 written by hand. Not re-checked |
| 16:346 | `war/clock.js` L16 | 180 | Derived: 24 hours / 8 = 3 hours = 180 minutes. L16 sets the factor to the travel scale (8, `world-scale.js` L45) |
| 16:379 | `docs/COF-IMPORT-MAP.md` L11 | 365, 180 | Arithmetic in the next sentence. L11 holds "the 8-year countdown to an alien armada" |

## 2. Lines fixed

Wrong numbers:

| Plan | Old | New | Source on the trunk |
|---|---|---|---|
| 00, 01, 04, 07, 17 | 1,047 cities | 1,109 cities (1,047 surveyed, 62 typed by hand and marked as estimates) | `src/data/cities.js` L1, L29 |
| 01 | 143 countries (168 nations in the country table) | 198 countries on both sheets | `src/data/cities.js` L1; `src/data/countries.js` L1, L15 |
| 00, 04, 01 (W0 pass line) | 168 countries | 198 countries (25 fields each, unchanged) | `src/data/countries.js`, `src/data/countries-30.js` |
| 01 (three lines) | 128 pedestrians | 80 pedestrians | `src/data/city-density.js` L22-23 (Robert, 2026-11-07: "I think crowds should be less") |
| 04 | 119 building types | 391 catalogue entries (88 are new generated-house entries) | `src/data/city-recipe-catalog.js` L41-53, counted in Node |
| 02, ledger J63 | 119 building types | 391 catalogue entries in 20 kinds (303 authored + 88 generated houses). Stage B5 and J63 now pass on: a scripted pass loads all 391, a scripted walk enters one building of each of the 20 kinds | same; kinds counted in Node |
| 09 | `hud.js` is 2,658 lines | 2,705 lines | `src/engine/hud.js` (line count) |
| 12 | Country relations 168 x 168 | 198 x 198 (168 authored, 30 derived by rule) | `src/data/relations.js` L391-393, `relations-30.js` |
| 16 | Superhero law: Regulated 101, Banned 39, Legal 28 | Regulated 129, Banned 41, Legal 28 | `src/data/countries.js`, counted in Node |
| 16 | Vigilantism: Banned 80, Regulated 77, Legal 11 | Banned 80, Regulated 104, Legal 14 | same |
| 16 | 14,028 pairs, 1,192 hostile, 11 allied | 19,503 pairs, 1,536 hostile, 13 allied (the old figures are Robert's own 168) | `src/data/relations.js`, counted in Node |
| 16 | Blocs: United Front 88, Collective 80 | United Front 110, Collective 88 | `src/data/relations.js` L31 |
| 16 | 805 treaty Ascendants across 168 teams | 835 across 198 teams (the 30 added states keep 1 each) | `src/data/treaty.js` L22-26, counted in Node |
| 16 | 161 countries have an army | 177 (jets 90, blue-water 44, submarines 22, carriers 12, power armour 12 are unchanged) | `src/data/military-branches.js`, counted in Node |
| 16 | Economy: 238 ms for all 1,047 cities; save 3.7 MB | Re-measured: 268 ms for all 1,109; 5.9 MB as plain JSON (old figures kept beside it, marked as the critic's) | `src/data/economy-sim.js`, run in Node on SOVEREIGN |
| 16 | 613,000 leader decisions in ten years (168 leaders) | over 720,000 (198 leaders) | arithmetic |
| 16 | One game day is 120 real minutes | 180 real minutes wherever a war runs; 120 kept as the time document's figure | `src/data/war/clock.js` L16 |
| 16 | Eight game years is 243 real days | 365 at 180 minutes a day (243 at 120, 61 at 30) | arithmetic |

Claims that were no longer true:

| Plan | Old | New | Source on the trunk |
|---|---|---|---|
| 16 | "What does not exist at all: ... wars between countries; any memory of what a country has decided ..." | A war layer exists; the section is rewritten | `src/data/war/` |
| 03 | Strength vs material ladder: "PROPOSED doc, nothing reads it" | Dated note: the force gate exists and is read | `src/data/force-gate.js` L1-37; `src/engine/city-session/damage.js` L24 |
| 03 | Punch against structures: (STR / 5) squared | The same, plus: a hit carrying strength-ladder power (1 to 12) uses that instead | `src/engine/city-session/damage.js` L22-27 |
| 08 | Ledge catch: "automatic" (a snap onto the top) | A real grab: hang 0.45 s, pull-up 1.4 s or 0.8 s, no catch when falling faster than 16 u/s | `src/engine/ledge-grab.js` L27-41; `src/data/heights.js` L25 |
| 12 | Police caps: only suspects bring the army | Plus: the state sends its army for its own reasons | `src/data/response-policy.js` L6-14; `src/engine/state-army.js` |
| 13 | Player save holds hero, cell, position, health, needs, custody | Plus death records, flying, time of day, nearest city cell | `src/engine/world-cell/player-save.js` L35 |
| 13 | Cell save holds damage, collapses, cars, items, households, economy | Plus tree damage, hung doors, objects placed in a home | `src/engine/world-cell/checkpoint.js` L69, L93; `src/data/world-registry.js` L42 |
| 01 | The browser game re-centres past 131,072 u | Kept (still true, `cell-stream-plan.js` L29), with a dated note: streaming now reaches 1,048,575 u and re-anchors on save | `src/engine/world-cell/cell-stream-plan.js` L35; `reanchor.js` |
| 14 | Plan 16 row: "Leaders of countries, starting as NPCs" | The trunk has a war layer and Unreal ports it | `docs/design/WAR-LAYER-2.md` |

Line references that had moved (the quoted numbers were re-read and are unchanged):

| Plan | File | Old lines | New lines |
|---|---|---|---|
| 03 | `city-session/damage.js` | L21-25 | L22-27 |
| 05, 08 | `shared-impact.js` | L8-31 | L6-33 |
| 05 | `game.js` carry speed | L1182 | L1191 |
| 05 | `game.js` throw speed | L1170 | L1179 |
| 05 | `game.js` throw arc and gravity | L1189-1205 | L1207-1232 |
| 05 | `game.js` object damage | L1205 | L1215 |
| 05 | `game.js` thrown body | L1329-1369 | L1339-1379 |
| 05 | `entity.js` into a wall | L1422-1428 | L1438-1444 |
| 06 | `entity.js` ki | L1417 | L1420 |
| 08 | `entity.js` base speed, slow-downs | L348, L2643-2697 | L353, L2681-2735 |
| 08 | `entity.js` take-off | L2076-2094 | L2094-2112 |
| 08 | `entity.js` power dive | L2171-2182 | L2189-2200 |
| 08 | `entity.js` hard landing | L2382-2388 | L2401-2407 |
| 08 | `entity.js` step-up | L2455 | L2474 |
| 08 | `entity.js` rise and sink | L210 | L212 |
| 08 | `entity.js` momentum | L2765-2789 | L2780-2816 |
| 08 | `entity.js` glider | L2237 | L2259 |
| 08 | `world.js` lock-on view | L2970 | L2978 |
| 10 | `abilities.js` type handlers | L383-1526 | L407-1576 |
| 10 | `entity.js` damage order | L978 | L983 (see the second-pass table) |
| 12 | `police.js` heat and stars | L52-54 | L54-56 |
| 16 | `power-elite.js`, `state-posture.js`, `fame.js`, `career.js`, `law-enforcement-ladder.js`, `treaty.js` | L65/L106, L38, L42-47, L76, L92, L8-18 | L3/L69, L39-41, L39-48, L77, L94, L32-33 |

Second pass, 2026-11-13, after the critic's review and the new script:

| Plan | What | Old | New (read on the trunk) |
|---|---|---|---|
| 10 | Damage order, `entity.js` | L984 | L983 (`takeDamage`), L1159 (armour bar 0.55), L1224 (frozen x1.3) |
| 08 | Momentum: drag 1.8, cap 210 | `entity.js` L2780-2816 | `core/util.js` L112, L114; the same drag at `entity.js` L211; the cap applied at `entity.js` L2723; coasting stays L2780-2816 |
| 08 | Landing flare 3.2, `data/flight-model.js` | L39 | L38 |
| 08 | Sneak x0.55 | `entity.js` L2681-2735 only | set in `engine/input-intent-live.js` L30, applied at `entity.js` L2689 |
| 08 | Glider air control x1.4, `entity.js` | L2259 | L2259 (fall cap), L2727 (x1.4) |
| 06 | Hunger under 15%, `data/needs.js` | L41-57 | L32 (the 0.15 line), L41-57 |
| 06 | Ki and needs, `entity.js` | L1420 | L1420 (spending ki), L1922 (needs cut ki regen) |
| 03 | Thrown body hurts above 30 u/s | `force-feel.js` L126 only | `entity.js` L1438-1444 added for the 30 |
| 12 | Six police levels | "L237-243", no file | `police.js` L239-244 |
| 16 | One round is 600 calendar seconds, `war-state.js` | L33 | L32; the 75 game seconds is `scenario-two-neighbours.js` L24-25 |
| 16 | Streaming reach 199 km, `cell-stream-plan.js` | L35 | L30-35 (199 is on L30) |
| 16 | The war layer's size | eleven files, 3,919 lines | eleven files, 3,178 lines (`wc -l`; equals the sum of the plan's own per-file table) |
| 16 | "The trunk's own ten open calls" | no source; the plan's table has 11 rows | ten numbered rows, `docs/design/WAR-LAYER-2.md` L196-207. The plan's table has those ten plus the War Room default (L190 of the same document), which is why it has eleven rows |
| 01 | `WORLD1-REPORT.md` (287,560 cells, 13.2 M records) | cited as a file | The file is not in the export. The citation is now plain words saying the two figures were not re-checked |
| 01, 16 | 15 line references with no file in front of them | bare "L45", "scenario L25", "(L30)" | each now names its file (`world-scale.js`, `vision-and-direction/GAME-DNA.md`, `scenario-two-neighbours.js`, `war-state.js`, `dossiers.js`, `operations.js`, `GLOBAL_TIME_OPTIONS_2026-09-18.md`, note 206), so the script checks them |

Checked and found unchanged (no edit): the unit and the storey height (`world-scale.js`; its only change is a comment);
28 cars, 2 buses, 22 live characters; 5 real buildings at once; 385 powers; 66 roster characters (the file's one change is
a field on one move); every carry, throw, wall-hit, guard, crouch, crawl, dive, landing, glide and lock-on number in
Plans 5 and 8; the body-contact rule (24 u/s, 14 + resilience x 3, 0.22, cap 28, 0.65 s); heat +12 and +40 and the six
star floors; unpowered suspects capped at level 4; player save version 1 and the 20 s autosave; the 8 MB cell cap; fame's
+4/+8 and +4/-10; alert odds 0.05 to 0.85; the Treaty's 1 to 20 and 11 at 20; the Combat Manual's section 60; the three
quoted lines of the design record and the two of the time document.

## 2b. Crowds (needs Robert's ruling)

The trunk dropped street pedestrians from 128 to 80 on Robert's word of 2026-11-07, "I think crowds should be less"
(`src/data/city-density.js` L22-23). It also has a new system, world people: `src/data/world-people.js` (the math),
`src/engine/world-cell/world-people.js` (the game hook), `docs/WORLD-PEOPLE.md`. A city simulates the square root of its
population as individual people (Miami 2,466, a village of 600 has 24), each with a home, a job, a personality, needs
and wants, stepped hour by hour; the bodies near the player are those people. Our plans targeted 2,000 anonymous people
and 300 cars. On 2026-11-13 that was changed everywhere it stood (Plan 12's people section, "This plan owns crowds"
paragraph, stages L3 and L5 and open questions; Plan 1's targets table and open question 2; Plan 14's Life row and
crowd-size call; Plan 7's Mass row; Plan 11's costume note; ledger J66; the charter's defaults table). The default now
reads: follow the game: a small number of individually simulated people near the player (the trunk's world people),
with the 2,000-figure crowd kept only as a measurement of what Unreal can afford, not a design target, until Robert
rules. It is marked in each place as needing his ruling.

## 3. New on the trunk that touches a plan

One line each: the file, what it is (from its own header), which plan it bears on and how.

| File (under `src/` unless it starts `docs/`) | What it is | Plan and effect |
|---|---|---|
| `data/war/*.js` (11 files), `docs/design/WAR-LAYER-2.md`, `docs/notes/206, 212, 213, 216` | War between countries as data and a headless simulation: clock, formations, journeys, knowledge, operations, files on people, the newspaper | 16: rewritten on top of it. 12: armies and checkpoints will come from it |
| `data/countries-30.js`, `data/relations-30.js`, `data/name-pools-30.js`, `docs/notes/201` | The 30 UN members the sheet lacked, with derived relations; every country now has a city | 1, 4, 16: all counts (198 countries, 1,109 cities). Exports must carry the "estimate" marks |
| `data/city-survey.js` | One answer for a city's crime and safety, including the 268 rows nobody surveyed (the country's estimate stands in) | 4, 12, 16: city values. Police response time and the economy now read it; export the survey answer, not the raw zeros |
| `data/state-posture.js`, `engine/state-army.js`, `data/world-event-rules.js` | The state's own army on the street: lockdown, a presidential order, an emergency; when street events become a world event | 12: a second way the army arrives, with checkpoints and curfews. 16: the alert-level lever already has its street side |
| `data/force-gate.js`, `docs/notes/179` | One rule of force against a threshold for everything hit hard; wielder power 1 to 12 from the strength ladder; vehicle armour | 3 and 10: the "proposed, unread" ladder now exists. Port with the damage rules |
| `engine/ledge-grab.js`, `docs/notes/197` | The ledge grab: catch the lip, hang, pull up | 8 and 5: the ledge row and course station change from a snap to a hang and pull-up. 11 (A5): the clips are `ClimbUp_2m` and `ClimbUp_1m` |
| `engine/stop-step.js`, `engine/gait-plant.js` | Feet plant when a run stops and during a sprint stance, instead of sliding | 8 and 11: foot-slide targets; the browser game's own slide figures in Plan 11 may be lower now (not re-measured) |
| `engine/grab-miss.js` | Names why a grab missed | 8, 10: a readable reason to carry over |
| `engine/carry-car-grip.js`, `engine/car-lift-motion.js`, `engine/carry-grip-sites.js` | Lifting a car as a clean-and-press with hands on the real sill, and the body under it | 5 (carry and throw), 11: the lift is no longer "rises straight up" |
| `engine/ragdoll-reflex.js`, `docs/notes/176-reflex, 177-ragdoll` | Ragdoll that protects and braces while physical | 8 (knock-back), 11 (A4) |
| `engine/modular-hit-flash.js` | The hit flash on the drawn body | 10, 9: hit feedback values (1, 0.5, 0.3, 0.22) |
| `engine/world-cell/reanchor.js`, changed `cell-stream-plan.js` | Streaming reach 199 km from the anchor; re-anchor on the nearest city on save and resume; visited cells let go past 96 | 1, 13, 16: the far-from-centre question and the save |
| `data/rolling-hills.js`, `data/hills-world.js`, `data/hills-world-relief.js`, `data/hills-old-saves.js` | Rolling hills under the whole travel layer, cities included, strength from real elevation | 1: the ground is no longer flat; the generator port must include them. Plan 1 does not mention hills |
| `data/country-road-plan.js`, `data/expressway-roads.js`, `engine/world-cell/expressway-stop.js`, `docs/COUNTRY-ROAD-MATH.md` | Expressways: two carriageways and a median, exits, rest stops with a building | 1 (W6 country) and 12 (vehicles): highways are more than one 64 u road |
| `data/country-woods.js`, `data/winter-cover.js`, `data/tree-wounds.js` | Woods by ecotype; snow by place and day; wounded trees in the save | 1 (open country, weather), 3, 13 |
| `data/us-state-cities.js` | Real cities added when the whole United States is planned | 1, 4: more places than the 1,109-row sheet in one country |
| `data/world-look-math.js`, `data/place-look.js`, `engine/world-cell/world-look.js`, `docs/WORLD-LOOK-MATH.md` | The planet's curve, haze by distance, the real sun, each region's air | 1 (far view), 6 (round Earth): the far view now drops with the curve at Earth / 8 |
| `data/world-people.js`, `engine/world-cell/world-people.js`, `engine/city-session/resident-people.js`, `docs/WORLD-PEOPLE.md` | Fewer people, each simulated with a personality, wants and a day; residents are those people | 12: the crowd plan (2,000 people) ran against Robert's 11-07 direction of fewer, meaningful people. Changed 2026-11-13 (section 2b); still needs his ruling before L3 |
| `data/death-rules.js`, `data/death-comeback.js`, `engine/death.js`, `engine/death-owners.js`, `engine/world-cell/clone-clinic.js`, `death-screen.js`, `morgue-desk.js`, `new-person.js`, `docs/notes/198, 214` | Death and the clone comeback: the body stays, who files the death, one clone life, come back as someone new | 13 (the player save now carries it), 9 (new screens), 10. No plan covers death; it needs a row in Plan 14's "not planned" table |
| `data/service/*.js`, `data/highwall-service.js`, `data/highwall-soldier.js`, `engine/highwall-service.js`, `docs/notes/211` | Recruiting, clone bays, the base armory, AWOL | 16 (where soldiers come from; the war layer's adapter), 12 |
| `data/giant-walkers.js`, `data/robot-units.js`, `engine/giant-walker-drive.js`, `engine/robot-squad.js`, `engine/highwall-robots.js`, `docs/notes/210` | Giant walkers, the spider tank, six robot units | 12 (vehicles), 14 (the "robots" row), 16 (units an army may field) |
| `data/bomb-bay.js`, `engine/bomb-bay.js`, `docs/notes/191` | Free-fall bombs from the heavy bomber | 12 (aircraft), 3 (blast), 16 (bomber runs wait on #1457) |
| `tool/vehicle-range.js` | A lab that fires every armed vehicle through the game's own fire path | 4 (tools), 15 and 12: the muzzle-position check to copy |
| `engine/world-cell/door-leaf-physics.js`, `hung-door-leaves.js`, `door-carry.js`, `docs/notes/194` | A broken door hangs on a real hinge, swings, can be knocked off; saved | 2, 3 (the first physics joint), 13 |
| `engine/world-cell/swat-entry.js`, `docs/notes/196` | SWAT stacks on a door and breaches it | 12 (police), 2 (doors) |
| `engine/world-cell/home-buy-mode.js`, `home-objects.js`, `docs/notes/192` | Buy mode: place, turn, move and lock objects in your own home; saved with the cell | 2 (furniture), 9 (a new screen), 13 |
| `data/gear-parts.js`, `tool/creator-gear.js`, `data/treaty-role-body.js`, `docs/notes/177-the-gear-library` | The gear library: reusable costume parts, outfits, the creator's gear panel | 11 (costumes) and 9 (creator): costume choice is now by named part and outfit |
| `core/sound-triggers.js`, `data/power-sound-slots.js`, `data/power-sound-bank.generated.js`, `data/arsenal-recordings.js`, `data/sound-clearance.js`, `data/sound-lab*.json`, `tool/sound-lab.js` | The Sound Pipe: every power, gadget and gun sound as a named slot Robert fills; burst rules for guns; licence receipts | 13 (sound) and 15 (gun sound): port the slots, not the code voices |
| `engine/collide-switch.js`, `bench/collision-audit.js`, `docs/reports/COLLISION_AUDIT_2026-11-02.md`; changed `pedestrians.js` | People, the player and cars no longer pass through each other | 12 (crowd), 5 (contact): the browser game now has person-to-person spacing; the "distances exported from `pedestrians.js`" in L3 include new ones |
| `engine/laptop-city-card.js`, `engine/title-close.js`, `data/country-words.js` | The laptop city tile; how the title closes; how a country is named in a sentence | 9 (screens), 16 (plain words) |
| `docs/notes/193-fracture-lab.md` | A fracture lab, not in the game: breaks from hit strength, loose pieces, joints | 3: evidence for the "what loses support falls" ruling |
| `docs/notes/195-engine-decisions...md`, `docs/TRAVEL-DESIGN-NOTES.md` | Engine decisions on sound through walls and carving; travel design notes | 13, 3, 6: reading before those stages |

Deleted on the trunk (they exist only on the side branch that was surveyed): `src/data/dominate.js`,
`src/data/grip-fit-ids.js`, `src/engine/grip-fit.js` and four notes on the bat hold and the Grip Lab. No plan cites them.
Plan 4's line "Grip Lab: nothing in the game reads its file today" is therefore true on the trunk as well.

Government structures: the trunk's own document says they are pull request #1458, **not merged**. Nothing for them is in
the export.

## 4. What changed in Plan 16

The banner "stale against the game's trunk" is gone. The plan now states that the browser game's war layer is the source
of truth and Unreal ports it. The table of what the browser game has was re-counted on the trunk (198 countries, 19,503
pairs, 835 treaty Ascendants) and gained the war layer. A new section describes that layer from its eleven files and
four notes, with its real numbers: the clock (calendar = play x 8, one war day is 3 real hours, rounds of 75 game
seconds), formations and units, journeys, what each country knows, the six operations and their three outcomes, the
newspaper, and the Germany against the Netherlands scenario (630 of 800). "What does not exist at all" became "What is
not in the browser game yet", which is now a short true list. "The war game" is in two parts: running a country as the
trunk defines it, with Robert's open calls and their defaults quoted; and the six accepted ideas the trunk does not have
(leader personalities, office through fame, the law and budget levers, countries remembering, the catch-up report,
server settings), each labelled, with the nearest thing the trunk has named beside it. "Time" is reconciled: the war
clock's 180 minutes a day against the time document's 120, with the reason the war's figure wins and a question for
Robert. Stages X0 to X5 now export golden results from the browser game and port the eleven files against them; X6
onward are kept, with X9 and X10 reworded to the trunk's paper and war room and a new X10b for the ideas not on the
trunk. The open questions dropped everything the trunk's document already asks. Kept as they were: how you play it and
the source-build finding, the two layers, the server in Unreal terms, Steam, the far-apart-players risk.

One finding for whoever runs X0: the war tools, tests and committed logs live in the trunk's `tools/` and `artifacts/`
folders, which `_lsw-master` does not carry, and the scenario file needs one of the game's packages to load. Ten of the
eleven war files load in plain Node from the export as it is.

## 5. Not checked

- **Changed files that no plan cites by path.** 254 of the 295 changed files. A plan may still quote a number from one
  without naming it in backticks. Only backticked paths were followed.
- **Plan numbers with no file beside them.** Not traced.
- **Files that did not change** between the checkout and the trunk were not re-opened, with these exceptions noticed on
  the way: Plan 4's "178 countries, 18,576 points" for borders and coast could not be reproduced (`borders.js` says 178
  countries and 21,135 points; the file is not in the diff list, so this is an older miss) and was **not fixed**; the
  count of "48 type handlers" in Plan 10 was not recounted (the change to `abilities.js` adds and removes no handler).
- **The surveyed checkout had 96 uncommitted files.** The comparison used its last commit, so a number a survey read
  from an uncommitted edit could differ from both sides of the comparison.
- **Line references in plans for files that changed but whose quoted numbers were only spot-checked**: `hud.js`
  (only its length), `pedestrians.js` (the ten states were not recounted), `audio.js` (the five channels were not
  recounted), `house-generator.js` (no numbers are quoted from it; it gained two optional inputs, front yard and door
  height), `country-grid.js`, `world-district.js`, `city-streaming.js`, `custody.js`, `grapnel-hook.js`,
  `sound-picks.json`, `treaty-teams.js`, `powers-roster.js` (385 confirmed, contents not compared).
- **The economy timings** are one run on one PC, three repeats, best taken. The size figure is plain JSON.
- **The war layer's results** (630 of 800 and the rest) are quoted from the trunk's notes. They were not re-run here:
  the tools are not in the export.
- **`docs/COMBAT_MANUAL.md`** gained 230 lines; only section 60 was checked.
- **Everything under `public/`, `tools/` and `tests/` of the browser game.** Not in the export and not in the diff list.
- **The 55 worktrees and other branches** of the browser game.
- **Plans R1 to R4** cite the engine, not the browser game, and were not re-checked, except R1's line on re-centring
  (still true).
