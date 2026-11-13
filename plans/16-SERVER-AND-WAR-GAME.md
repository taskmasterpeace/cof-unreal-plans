# Plan 16: The server and the war game

Status: PLAN ONLY (2026-11-12; reconciled with the browser game's trunk 2026-11-13, job J04). Nothing here is built in
Unreal.

Robert's ruling, 2026-11-12: the Project Zomboid model (solo, friends, community servers), built as two layers, with
official war seasons added later, and all the war game ideas. A hero stays on the server where it was made.

**The browser game's war layer is the source of truth and Unreal ports it**, the same way the controller's rules are
ported: every rule is copied into engine-free code, every number comes from a named line on the trunk, and the port is
checked against results exported from the browser game. This plan was first written from a checkout 515 changes behind
the trunk and said no war layer existed. That was wrong. Sections "What the browser game already has", "What is not in
the browser game yet" and "The war game" below were rewritten on 2026-11-13 from the trunk export (`_lsw-master`, commit
`c733c398`). What changed is listed in `R5-TRUNK-RECONCILE.md`.

Reviewed by the critic on 2026-11-12: 28 of 34 facts confirmed, 4 wrong, 2 bad citations; verdict pass with fixes. That
review covered the first version. **The rewritten sections have not been read by a critic.** Two of its findings still
matter: a stand-alone server needs a different build of the engine than the one installed, and players in far-apart
cities have no ready-made answer in Unreal.

This is not new scope. The game's design record
(`docs/hub/10-game-design/vision-and-direction/GAME-DNA.md`) says: "If we could have the level of Project Zomboid, the way
time goes" (`vision-and-direction/GAME-DNA.md` L113), "He always imagined LSWs working for governments, with missiles, 'a war game.'" (`vision-and-direction/GAME-DNA.md` L168) and "The
country/war layer is real DNA, not scope creep." (`vision-and-direction/GAME-DNA.md` L169). Robert's ask of 2026-11-12, which the trunk's war layer 2 is
built from: "if you can control a country ... it becomes a war game in a sense ... We need to have covert ops,
kidnapping, assassination. So you got to know who people are ... a part of me thinks we need some type of newspaper"
(`docs/design/WAR-LAYER-2.md` L9-13).

## How you play it

| Way to play | What it is | Who runs the server | Works from today's engine install? |
|---|---|---|---|
| Solo | Your own world. Your PC runs the server inside the game | You, without noticing | Yes |
| With friends | You host; friends join you | You | Yes |
| Community server | A server with its own settings, running day and night with no screen | Whoever sets it up | **No. Needs the engine built from source** |
| War season (later) | An official server running one war start to finish | Us | Same |

**The source-build finding.** The Unreal on SOVEREIGN is the launcher version. It can build the game and a game that
hosts, but not a screenless server program; the engine refuses with "targets are not currently supported from this
engine distribution" (checked in the install). Epic gives the engine's source free to anyone with an Epic account linked
to GitHub. Building it is a long one-time compile and takes a lot of disk. It is not needed until community servers, so
it is scheduled late, but it is a known step, not a surprise.

Each server is its own War World. A hero lives on the server where it was made.

## Two layers

| | The strategy layer | The action layer |
|---|---|---|
| What it holds | 198 countries, their leaders, laws, budgets, who is at war, armies as named people and vehicles, what each country has seen, the economy, fame, the news | Streets, buildings, people, fights, destruction |
| Where it runs | Everywhere, all the time, as numbers | Only in the city cells near a player |
| What it costs | Very little (measured below) | Nearly everything |
| How it is built | Engine-free rules with tests, ported from the browser game like the controller | Unreal |

When a player enters a city, the action layer builds it from the seed (Plan 1) and dresses it from the strategy layer's
current numbers. When the last player leaves, the city goes back to being numbers, and what happened there is written
into them. The trunk's war layer names this same seam and has not built it: "a formation near the player has to become
real vehicles and people ... Not built. Every vehicle and person has an id and a place, so the data is ready for it"
(`docs/notes/206-war-layer-formations.md` L464-465).

**How cheap.** The economy model (`src/data/economy-sim.js`), measured by the critic on the old checkout: twelve game
months took 17 thousandths of a second for 60 cities and 238 for all 1,047; save 220 KB and 3.7 MB. Measured again on
2026-11-13 on SOVEREIGN against the trunk, which has 1,109 cities: 26 to 30 thousandths of a second for 60 cities and
268 for all 1,109; the state written as plain JSON is 0.33 MB and 5.9 MB (a different way of measuring size than the
critic's, so the two size figures are not comparable). The war layer: the trunk plays 800 three-day wars, 345,600 ticks,
in "about five minutes" in Node (note 206 L22), which is under a thousandth of a second a tick for a two-country war.
Not measured: a war layer with more than two countries in it.

## What the browser game already has for the strategy layer

Counts are from the trunk, re-counted in Node on 2026-11-13.

| Piece | State | Where |
|---|---|---|
| 198 countries, 25 fields each: government type (26 kinds), how clean the government is, military, intelligence and police budgets, press freedom, wealth, healthcare, science, terrorism, how normal powered people are, and the laws below. 168 are Robert's sheet; 30 small states were added and their guessed fields are marked as estimates | Data | `src/data/countries.js`, `src/data/countries-30.js` |
| Superhero law per country: Regulated 129, Banned 41, Legal 28. Vigilantism: Regulated 104, Banned 80, Legal 14 | Data | same |
| A leader for every country, generated from its row (title, name, seat in the country's largest city). 3 are written by hand in code (Greece, Venezuela, Haiti); the lore names 11. Each country also has oligarchs with "a direct line" to the leader | Built | `src/data/power-elite.js` L3, L69 |
| How every pair of countries gets along, 1 to 5: 19,503 pairs, 1,536 hostile, 13 fully allied. Of those, 14,028 pairs (1,192 hostile, 11 allied) are Robert's own matrix and the rest are derived by rule for the 30 added countries. Two blocs: United Front 110, Collective 88. The table itself is fixed; the war layer lays a per-world change on top (a grudge, below) | Data | `src/data/relations.js`, `relations-30.js` |
| The Treaty: every country may keep between 1 and 20 Ascendants; the 11 most populous keep 20; the 30 added small states keep 1 each. 835 are named across 198 teams | Data | `treaty.js` L32-33, `treaty-teams.js` |
| Military by country: 177 have an army, 90 fast jets, 44 a blue-water navy, 22 submarines, 12 carriers, 12 power armour | Data | `military-branches.js` |
| City alert level rolled daily: normal, alert, lockdown, martial law, with odds from the kind of government (0.05 to 0.85). On the trunk the army then really deploys on the street: checkpoints, patrols, a curfew | Built | `state-posture.js` L39-41, `src/engine/state-army.js` |
| Fame per hero per country: Known 0-100, Regard -100 to 100. A rescue is +4 known, +8 regard; harm is +4, -10. Fades daily | Built | `fame.js` L39-48 |
| Government contracts for the player's firm, by how two countries stand | Built | `career.js` L77 |
| The economy model with treasuries and shocks: war, embargo, blockade, harvest, strike, pandemic, disaster, aid | Prototype | `economy-sim.js` |
| Ten living-world systems: rumours that change as they spread, chains of consequence, promises, legacy after death. They work per person at city scale | Built and proven headless; **switched off until Robert rules** | `src/data/living-world/`, switch at `src/engine/world-cell/living-world.js` L90 |
| News written from events (KMK 9) | Built for fights only | `src/data/news.js`, `src/engine/newscrew.js` |
| Personalities: 20, with 7 drives (dominance, safety, duty, glory, vengeance, order, purpose) | Built for fighters | `psyche.js` |
| **The war layer: war between two countries, with a clock, formations, journeys, knowledge, covert operations, files on people and a newspaper** | **Data and a headless simulation. Not in the running game** | `src/data/war/` (eleven files, 3,178 lines), described next |

## The war layer on the trunk (what Unreal ports)

Built from Robert's rulings (note 206 L35-41) and his ask of 2026-11-12. It is pure data and rules: no drawing, no engine,
no random numbers outside the world's own seeded cursor, no wall clock. Ten of its eleven files load in plain Node from
the export (checked 2026-11-13); the scenario file also reads the city map record, which reaches a package the export
does not carry.

| File | Lines | What it holds |
|---|---|---|
| `clock.js` | 22 | The one clock |
| `capability.js` | 129 | The country sheet turned into what a country can organise: one sheet number feeds one channel, never a damage bonus |
| `units.js` | 156 | 26 units, each an id from the game's fleet, with which columns are from the game and which are a hand table |
| `formations.js` | 765 | The world: places, roads, bases, named people, vehicles, formations, kit, piles left on the ground, the supply ledger, wounds and aid posts, the audit, save and load (save format 3) |
| `war/movement.js` | 269 | Routes and journeys. The only way anything changes place |
| `war/knowledge.js` | 418 | What each country has seen (its "book"), watchers, late reports, the news, the country's objective and its planner |
| `operations.js` | 414 | Six kinds of covert operation, the three outcomes, evidence, prisoners, consequences |
| `dossiers.js` | 160 | A country's files on people, read out of its book |
| `war-state.js` | 515 | Declaring war, the fight, air strikes, eating, supply, occupation of districts, and `stepWorld`, the one tick |
| `newspaper.js` | 76 | Each country's paper, printed from what the news carried |
| `scenario-two-neighbours.js` | 254 | Germany against the Netherlands, built from the sheet |

### The clock and its compression

- One named factor: calendar seconds = game seconds x 8 (`clock.js` L16, equal to the travel scale, `world-scale.js` L45).
  A real kilometre is one eighth of a game kilometre and vehicles drive at their true speed, so one hour of play is eight
  hours of the war's calendar. **One war day is 3 real hours.**
- The factor is applied in exactly one place, `stepWorld` (`war-state.js` L484). Movement, fuel, rations, late reports,
  treatment, repair, the call-up and the news all count on that calendar.
- One round of fighting is 600 calendar seconds, which is 75 game seconds (`war-state.js` L32, `scenario-two-neighbours.js` L24-25). Columns on
  the road are re-read every calendar minute (`war/movement.js` L128).
- **Any tick that is a whole multiple of 75 game seconds plays the identical war, byte for byte. A shorter tick does
  not** (note 206 L64-70). So the server must step the war in whole rounds.
- A scenario war lasts 3 calendar days: 9 hours of play, 432 ticks. When the capital fell, the middle war took 34 hours
  of the war's calendar, a little over 4 hours of play (note 206 L76-77).

### Formations and units

- A formation is named people, vehicles with crews, and counted supplies (fuel, ammunition, rations) and kit (radio,
  charges, tools, dressings). 26 units; the classes and what each budget step allows are hand tables
  (`capability.js` L47-69: for example 12 tanks, 30 trucks and 6 jets allowed per step; call-up of reserves in 96, 72,
  48, 24 or 12 hours by service step).
- A country's forces come from its sheet row and its standing among all countries, so they moved when the sheet grew to
  198 rows. In the scenario Germany fields 26 vehicles and 129 riflemen in the column and 13 and 129 at home; the
  Netherlands 11 and 31 in the capital and 9 and 15 forward (note 206 L186-188). One rifleman on the map stands for a
  thousand (`scenario-two-neighbours.js` L46).
- The conservation laws, each with deliberate-break checks: supplies are made once and then only move or burn; a person
  and a vehicle are each in exactly one place; the dead stay dead; nothing moves without a mover (note 206 L128-147).
- The fight: rounds are simultaneous; a country's name decides nothing; hit chance comes from punch against armour (0.6,
  0.5, 0.3, 0.1, 0.005); dug-in troops at their own garrison are harder to hit (0.6, after 20 minutes)
  (`war-state.js` L26-36).
- Holding a city is by district: an unopposed army takes one district an hour for every 12 soldiers; the holder of a
  city is the army with more than half its districts (`war-state.js` L39, note 206 L149-159).

### Journeys

Nothing teleports (ruling 5). A journey covers no more than its speed times the calendar time given and no further than
the fuel in its tanks; a vehicle that runs dry or has no crew is left on the road as a pile that stays in the world.
Enemy columns that come within 5 km halt and fight there (`war/movement.js` L122). A wing flies only from a working base with
an open runway and fuel for the round trip. People march at 9 units a second and carry 20 kg (`units.js` L21-23).

### What each country knows

- A country acts only on what it observed (ruling 4). Its book holds facts with a source and an age. Its planner
  (`decide`, `war/knowledge.js` L342) is handed the book and the country's own forces and nothing else, and every order that
  is not part of the opening plan names the report it answers.
- Watchers are people in a place: a scout post sees 12 km, a radar 60 km, aircraft 15 km (`war/knowledge.js` L19). How many
  a country has comes from its intelligence budget step (1 to 5 scouts, 0 to 3 informants, `capability.js` L55-61).
- A report reaches the book at once with a working signals post and 3 hours late without; a sighting is fresh for 2
  hours; the news reaches every country 1 hour later (`war/knowledge.js` L21-25).
- Files on people (`dossiers.js`): only heads of state, treaty members, operatives and formation commanders are filed.
  Each line says where it came from and how old it is. How sure: the public record is "evidence", our own people saw
  them is "known", a prisoner's word is "suspected" (`dossiers.js` L30). A file never follows the truth: a person who moves is still
  filed at the old place until seen again.

### Operations and their three outcomes

- Six kinds: recon, kidnap, assassinate, sabotage, extract (rescue), capture a living superweapon (`operations.js` L31).
- A team is named people who leave a real base by road with counted kit. There is no per-day limit: an operation needs a
  fit team that has rested 8 hours, its kit, a vehicle and fuel.
- **Three answers that do not depend on each other** (ruling 8): was the objective done; who escaped, was captured or
  was killed; what evidence was left and whether it points at the sender. Base chances of the objective: recon 0.85,
  sabotage 0.7, extract 0.6, kidnap 0.55, assassinate 0.5, capture 0.35; of being noticed 0.15 to 0.75
  (`operations.js` L38-57). Being noticed does not name the sender: that needs evidence that points and a working
  analyst.
- A kidnap or killing can be ordered only against a person in the sender's files, and goes to the place the file says
  (`canLaunch`, `operations.js` L111). A refused launch leaves the world byte for byte as it was.
- A country's planner hunts a person only when its own people saw them within the last day, and the person is the
  enemy's leader, a treaty member, or a commander within 80 km of its own troops, and is not a story character
  (`war/knowledge.js` L341).
- Consequences (`operations.js` L71): a commander killed or taken leaves his formation without orders for 2 calendar
  hours; a leader killed or taken leaves the country unable to sign a declaration or a call-up for 1 calendar day; when
  evidence names the sender, the hit country's relation with it drops one step, never below hostile. War may be
  declared at relation 2 or below (`war-state.js` L37). Operations need no war.

### The newspaper

Each country has its own paper with its own name ("The Amsterdam Standard"). It prints what became public, never the
truth: an attack nobody saw is in no paper, and a story never says who sent a team until evidence names them. One
edition each war morning at 06:00, and an extra the moment a capital falls or a leader is taken or killed. Press freedom
from the country sheet decides what a paper prints about its own losses: 60 or more prints them the day they are known;
34 to 59 a day late and small; under 34 never (`newspaper.js` L20-21, note 216). Not built on the trunk: a battle's own
story, prisoners paraded or exchanged, a body found later, and reading the paper in the running game.

### The scenario and what it shows

Germany against the Netherlands, 800 seeded wars in four sets of 200, 3 calendar days each, nobody steering
(note 206 L214-263, and note 212 for the prisoner line):

| Result | Number |
|---|---|
| Germany took Amsterdam | 630 of 800; set by set 157, 155, 151, 167 |
| The Netherlands held to the deadline | 170 of 800; set by set 43, 45, 49, 33 |
| Noise band (two counts this close cannot be told from chance) | 16 wars in 200 |
| Ticks played, laws broken | 345,600 and 0 |
| Orders given, of which reactions to a fact in the country's own book | 32,859 and 26,030 |
| Raids finished | 6,412 |
| Without its supply column, Germany took Amsterdam | 0 of 800 |
| With German informants posted in Dutch cities (the "spies" what-if, note 213) | 726 of 800; set by set 184, 174, 182, 186 |

The trunk keeps these as committed logs (`artifacts/war-sim/*.txt`, six files) and a test replays the 800 wars and fails
if a line differs. Tests on the trunk: ten suites and 214 tests for slice one (note 206 L19-20), plus 21 for files on
people, 18 for kidnap and kill and 16 for the paper; 199 deliberate breaks, each of which must be caught
(120 + 24 + 28 + 27). **The tools, tests and logs are in the trunk's `tools/` and `artifacts/` folders, which the export
copy `_lsw-master` does not carry yet.** Stage X0 adds them.

## What is not in the browser game yet

Checked against `docs/design/WAR-LAYER-2.md` and the eleven files on 2026-11-13.

| Thing | State on the trunk |
|---|---|
| A war in the running game | None. Nothing in the game imports `src/data/war/`. The war room, the war map and the paper app are part A, not started |
| More than one war, or more than two countries | The one scenario. The rules take any countries; no other map of places and roads exists |
| A clock that runs when nobody is playing | None. The war steps only when something calls `stepWorld` |
| Government offices and who orders the army | Pull request #1458, **not merged**. Until then the head of state stands in |
| Elections, coups, a successor by name | None. A killed leader is replaced after a day by "the government", unnamed |
| Relations that change in peacetime | Only the grudge step from a named covert operation |
| Leader personalities, law and budget levers, countries remembering for years, a catch-up report, server settings, seasons | None. See "The owner's accepted ideas" below |
| Peace | "Sue for peace" is named as an order door in part A; there is no peace rule in the eleven files |
| Sea and air war beyond strikes | Artillery, crewed anti-air, bomber runs, the gunboat and AI pilots wait on pull request #1457 (`units.js` L145) |

The browser game's online code is two players in a room with nothing saved, and its server stores saves but simulates
nothing. That part of the first version of this plan still stands.

## The war game

Two parts: what the trunk has defined and Unreal ports, and Robert's accepted ideas that the trunk does not have yet.

### Running a country (the trunk's part A; defined, not built)

From `docs/design/WAR-LAYER-2.md` L170-190. The player who controls a country gets the same doors the country's own
planner uses, and no more: move a formation, send a sortie or an air strike, launch a covert operation, declare war,
sue for peace. They see only their country's book and its files, on a war map. Time runs on the war calendar and the
player can pause or speed it. An order the planner's door refuses is refused for the player too, with the same reason.

Robert's open calls there, with the default the browser game builds until he answers (quoted):

| Question | Default (ROBERT) |
|---|---|
| How does a player come to control a country? | "A 'lead a country' start, picked when you start a war game." Rising to lead from the career "comes later" |
| Where is the War Room? | "a screen on the laptop (L), opened from a war-room app" |
| Killing a commander: how long is the unit without orders? | "2 calendar hours, then the next senior member takes over." |
| Killing a country's leader: what happens? | "No new war orders for 1 calendar day, then the successor from the order chain (#1458)." |
| Does a killing stick in a clone world? | "It sticks for this war: a clone comes back after the war's three days at the earliest" |
| Can a living superweapon take power after a leader is killed? | "Not by the dice; only as a written story event." |
| What happens to a kidnapped prisoner when the war ends? | "Exchanged at the peace, one for one; the rest go home." |
| What are the papers called? | "Each country's paper has its own name; THE SIGNAL is the app and wire that carries them." |
| Can canon characters be kidnapped or killed? | "A country's own planner never picks them; a player's order against one is allowed, but a killing becomes 'wounded, got away' and a kidnap fails" |
| Are treaty members publicly known? | "By codename; the real name is secret." |
| Can the player go on a kidnapping themself, in the 3D world? | "Later, when the war reaches the engine"; until then the dice decide |

Unreal ports these defaults as they stand. The last one is where Unreal adds something the browser game cannot yet: the
operation's result can be handed in from play (`resolveOp` takes a `playerResult`, `operations.js` L235), so a kidnap
fought in the action layer replaces the dice. That is stage X6.

### The owner's accepted ideas that are not in the browser game yet

Robert accepted these on 2026-11-12. None is on the trunk. Each stays in this plan, to be built as engine-free rules
with tests after the port, on top of the trunk's doors and never beside them. Where the trunk has a nearer thing, it is
named.

**1. NPC leaders with personalities making decisions. Not in the browser game yet.**
The trunk's country planner (`decide`) is one written rule for every country: it reads the book and reacts. It has no
personality. The idea: every country's leader (the 11 the lore names, the rest generated) gets a personality from the
existing 20 and weights on the seven drives, re-read at country scale: dominance wants territory and tribute, safety
wants allies and a strong army, order wants strict superhero law, glory wants to win wars, vengeance remembers who
wronged the country. The weights choose among orders the planner's doors already allow; they never add a door and never
let a leader act on something not in the book.
- A leader acts at most once a game week on matters of state (law, budget, relations). Without that limit, 198 leaders
  acting daily would be over 720,000 decisions in ten game years and the map would never settle. War orders stay on the
  war layer's own tick.
- Leaders fall: lost elections in democracies (on a fixed term), removal by the oligarchs in hybrid states, coups in
  authoritarian ones, assassination (which the trunk has), death in a war. These wait on the government work (#1458) for
  who succeeds.

**2. Players becoming leaders through fame. Not in the browser game yet.**
The trunk's default is a "lead a country" start; rising from the career "comes later". The idea for later:
1. Be known and liked there. A threshold on each opens the door (a starting guess: Known 70, Regard 50).
2. The route depends on the government the country row names: win an election, be appointed by the oligarchs, or take it
   by force.
3. One seat, several candidates: in an election the higher Regard wins, with ties broken by Known; by force, the last
   one standing.
4. Holding office is a real job. A leader who is offline is run by the country's own planner, which is exactly what the
   trunk's part A does when no player holds the country. It runs only while the server's clock runs, so on a paused solo
   world nothing happens behind your back.
5. A leader and a citizen in the same country are both bound by the law as it stands. A leader cannot exempt themselves.

**3. The leader's levers: superhero law, vigilante law, budget, alert level. Not in the browser game yet.**
On the trunk a country's player can move forces, strike, run covert operations and declare war. The sheet's law and
budget fields are read by the war layer but nothing can change them; `lswRegs` and `lswActivity` are "defined, not used
yet" (note 206 L92-93).

| Lever | Effect in the strategy layer | What players see on the street |
|---|---|---|
| Superhero law: legal, regulated, banned | Changes the country's row in a per-world layer, the way a grudge changes a relation | Police ignore, check or hunt powered people; the player's firm becomes legal or not (already wired to this field) |
| Vigilantism: legal, regulated, banned | Same | Whether stopping a crime earns thanks or a warrant |
| The budget: police, military, intelligence, healthcare, science | Moves the country along the war layer's own ladders (vehicles allowed, watchers, call-up time, hours to heal), which are steps, not sliders. Paid from the treasury in the economy model | How fast and how hard the six police levels arrive (Plan 12); the army the country can field |
| Alert level | Sets the posture the trunk already rolls daily and deploys | Curfews, checkpoints, martial law (built on the trunk, `state-army.js`) |
| Relations: improve, sour, ally, embargo | One step at a time in the same per-world layer the grudge uses; a pair cannot move again for 30 game days. War itself is the trunk's `declareWar`, allowed at relation 2 or below | Prices, travel, which contracts are offered, armies in cities |
| The country's own Ascendant team | On the trunk a country with a treaty team has it at its garrison and its planner sends it where its troops report the enemy. At home it is the top rung of the law where superhero law is not Banned and powered activity is 40 or more (`law-enforcement-ladder.js` L94) | A named hero team comes for you |
| Contracts | Offers work to player firms | The job board |

How the last two fit the lore: the Threshold Treaty caps how many powered people a government may keep and bars it from
using more directly. The country's team is its legal allowance. Anything beyond that, and anything abroad, goes through
companies. That is why the player runs a firm, and it is how a player serves or defies a leader without being one.

A law or alert change never lands mid-fight: it takes effect at the next game midnight, and police already engaged
finish under the rule they started with.

**4. Countries remembering. Partly on the trunk.**
What the trunk has: each country's book (every fact with its source and age), its files on people, the grudges it bears
(`world.grudges`) and every news item ever carried (`world.newsArchive`), all kept through a save. What it does not have:
memory that outlasts one war, or that a leader's choices read. The idea: each country keeps a history of what was done to
it and by whom, capped, which the vengeance drive reads. Destruction, rescues and harm in a city are written into the
strategy layer as they happen, in small batches every few seconds, so a crash loses at most the last batch.

**5. The catch-up report. Partly on the trunk.**
What the trunk has: the newspaper. Old editions stay readable after a save and load, so a returning player can read the
mornings they missed, in their own country's paper, with that paper's bias. What it does not have: one report across
everything that changed (leaders, laws, borders) in the countries the hero is known in. The idea stands, built from the
news archive and the history, and it must never show more than the papers could have printed.

**6. Zomboid-style server settings. Not in the browser game yet.** See "Server settings" below.

### What heroes do changes the map

- The trunk's seam: a formation near a player becomes real vehicles and people, and what happens to them is written back
  to the same ids (stage X6). A covert operation's result can come from play.
- A country reacts through its planner on the next war tick, and (when idea 1 exists) its leader reacts on matters of
  state at the next weekly turn.
- Fame is already per hero per country and already booked from rescues and harm; it is ported as it is.

### Time

| Rule | Source |
|---|---|
| One war day is 180 real minutes: the war clock's factor of 8, tied to the travel scale | `src/data/war/clock.js` L16 |
| The game's time document recommends 120 real minutes a game day, with presets 30, 60, 120, 240 and 1,440 | `GLOBAL_TIME_OPTIONS_2026-09-18.md` L72 |
| Solo and friends: time stops when the server stops. A community server never stops | "Pause by default for co-op; continue for persistent worlds", `GLOBAL_TIME_OPTIONS_2026-09-18.md` L76 |
| Sleeping never skips time for other players | `GLOBAL_TIME_OPTIONS_2026-09-18.md` L87 |
| Catch-up when you return | The newspaper's back editions; later the catch-up report |

**Reconciled with the trunk's war clock, 2026-11-13.** The two sources disagree: the time document says 120 minutes a
day and the war layer, built later, runs 180. They are not the same kind of number. The war's 8 is not a taste setting:
it is what makes a tank at its true speed cover the right real distance on ground laid out at one eighth scale. A day of
120 minutes on that ground would make every journey half as long again on the calendar as its real distance says.
So this plan takes the war clock as the truth: **the default day is 180 real minutes wherever a war can run**, the
port uses 8 so its results can be checked against the browser game, and the day-length server setting is the war's own
factor (`createWorld({ compression })`; the trunk's clock test changes it and proves every duration moves together).
A server that changes it gets a war that is no longer the tested one, and the settings screen says so. Whether 180 or 120
is the default is asked of Robert below.

The layer's clocks: the war steps every 75 game seconds in whole rounds; fame and city alert level step daily; leaders
(idea 1) weekly; the economy monthly; elections on their term. One clock drives all of them. A paper comes out at 06:00
on the war calendar.

### Server settings (like Zomboid's sandbox)

Not in the browser game yet. Day length (the war factor, above). How harsh the needs are. Whether destruction is
permanent or repairs over time. Police aggression. A cap on power level. Whether players can lead countries, and by
which route. Whether NPC leaders have personalities or all use the plain planner. Whether the living-world systems are
on. How often leaders act. Player limit. Whether new heroes can join mid-war. Each setting is a number or a switch in
the world state, saved with it, and each has a test that it changes its number.

### War seasons (later)

Not in the browser game yet. An official server runs one war with a start, an end and a winner, then a new world begins.
The trunk's scenario already has that shape in small: a deadline of 3 calendar days and a stated objective for each side
("hold the capital until the deadline"). The lore offers a longer clock: the countries arm against each other during an
eight-year countdown to an alien armada (`docs/COF-IMPORT-MAP.md` L11). Eight game years is about 365 real days at 180
minutes a game day, 243 at 120, and 61 at 30. Season length and what winning means are Robert's to set.

## The server, in Unreal terms

- **Solo and friends** use a game that hosts, which the installed engine supports. **A screenless server** needs the
  source build (above).
- The server owns the seed, the clock, the strategy layer (the war world included), every hero, and the list of changes
  to the world.
- **Each player's client is sent only its own country's book, files and paper.** The trunk's rule that a country acts
  only on what it observed becomes a rule about what the server sends: the truth record never leaves the server.
- **Who can cheat.** On a server someone else hosts, players cannot edit their heroes, because the files are not on
  their PC. The host always can. That is the same as Zomboid, and it is why official season servers exist.
- **Same city on every PC.** Each PC builds the city from the seed, so the generator must give the same answer on
  different hardware. The rule file therefore uses whole numbers or fixed-order arithmetic for anything that decides
  where a wall is, and the test builds the 6 reference cities on at least two different PCs and compares them exactly.
  The war layer has the same need, and the trunk already meets it in JavaScript: the same seed gives the same war, and
  every random draw walks its table in id order.
- **Players in far-apart cities: the main risk.** Unreal expects one loaded area per server. Its own tools for this are
  switched off or unfinished in 5.9 (server streaming for hand-made maps is off by default and we do not use those maps;
  the newer replication systems are Beta; the multi-server one is Experimental). Our own cell streamer keeps cells alive
  around each player, and the server sends each player only what is near them. That part is ours to build.
- **This collides with a fallback in Plan 1.** Re-centring the world on the player works for one player; a server
  cannot re-centre on several players in different cities at once. So for server play the far-from-centre test (Plan 6,
  S5) stops being optional: either fights work correctly hundreds of kilometres from the centre, or each active city
  must be run in its own space on the server. Stage X8 decides, and it is pulled forward. (The trunk itself now streams
  out to 199 km from its anchor and re-anchors on save: `cell-stream-plan.js` L30-35, `world-cell/reanchor.js`. That is
  still one player.)

## Saves

The strategy layer's file follows Plan 13's rules: its own versioned file beside the world's, written off the main
thread. The war world carries the trunk's own save format number (3, `formations.js` L24) and its own rules: a save in
the middle of a war continues identically, and an old save that cannot say who held a town is refused, not guessed.
Budget: economy 5.9 MB at full size as plain JSON (measured 2026-11-13; smaller packed); the war world's size is not
measured yet and X5 measures it; history capped at the last 200 entries per country plus one summary line per country
per game year; leaders and relations under 1 MB. The news archive grows with every story and needs the same kind of cap
on a server that never stops. Block damage on a server that never stops grows without limit unless it is managed, so the
"destruction repairs over time" setting doubles as the size control: repaired cells drop out of the save.

## Order

The paper stages port the trunk's war layer. Each is checked against results exported from the browser game ("golden
cases"), the same method as the controller. "Identical" means: every count, every id, every place and every state
equal; positions and amounts equal after rounding to six decimal places.

| # | Stage | Needs Unreal? | Done when |
|---|---|---|---|
| X0 | **Golden export.** Extend the trunk export with the war tools, their helpers and the six committed logs (`tools/war-*`, `tools/helpers/`, `artifacts/war-sim/`), and the game's packages inside the export copy, so the scenario runs. Write golden cases as JSON under `Data/war/`: the capability of all 198 countries; the 26 units; and for seeds 1 to 40 the full world state at ticks 0, 108, 216, 324 and 432 plus a hash of it at every tick | No | Run from the export, `tools/war-sim.mjs 200` prints a log identical to the committed `two-neighbours-4x200.txt`, line for line (Germany 630: 157, 155, 151, 167); the trunk's war test suites all pass there and their test count is written in the proof; the golden folder holds 198 capability rows, 26 units, 40 x 5 states and 40 x 433 hashes; counts in the export read 198 countries x 25 fields, 19,503 pairs, 835 treaty Ascendants |
| X1 | **The tables.** Port `clock.js`, `capability.js` and `units.js` to engine-free code | No | Capability equals the export for all 198 countries on every channel, and the 26 units on every column: 0 differences. Changing one sheet number moves only its own channel (the trunk's test, re-run on the port). Changing the factor from 8 to 4 halves every calendar duration |
| X2 | **The world and movement.** Port `formations.js` (save format 3, the audit, the supply ledger) and `war/movement.js` | No | The 200 exported states load and re-save identical to the export; the audit reports 0 broken laws on each; a scripted journey of the German column from Duesseldorf to Utrecht with no enemy matches the export's position at every tick; a refused order leaves the state hash unchanged |
| X3 | **Knowing and acting.** Port `war/knowledge.js`, `dossiers.js` and `operations.js` | No | For seeds 1 to 40, each country's book and files at ticks 108, 216, 324 and 432 are identical to the export; Germany's files at hour 0 are the Dutch head of state at Amsterdam and the 3 Dutch treaty members and no one else; a kidnap ordered against a person with no file is refused with the trunk's reason and the state hash is unchanged |
| X4 | **The whole war.** Port `war-state.js`, `newspaper.js` and the scenario; play it | No | Seeds 1 to 40: the state hash equals the export at all 433 ticks. All 800 wars: Germany takes Amsterdam in 630 (157, 155, 151, 167), 0 broken laws in 345,600 ticks, and the port's log equals the committed one line for line. The spies run gives 726 (184, 174, 182, 186). Ticks of 150, 300 and 600 game seconds give the same wars as 75 for seeds 1 to 40. Both papers of war 1 on its last morning equal the export's text |
| X5 | **The rest of the numbers, and cost.** Port the economy model, city alert level and fame; measure the war | No | A 12-month economy run matches the browser game's ledger within 0.01% on every balance for the 60-city sample; fame matches exactly for a scripted list of 50 events; the 800 wars play in under 300 s on SOVEREIGN (the browser game's own time in Node); one war tick is under 5 ms at worst; a mid-war save is under 2 MB; the strategy save without the war is under 8 MB |
| X6 | City in, city out | Yes; needs Plan 1 through its streamer stage and Plan 12's police | Raise a country's alert level in the strategy layer and the city spawns with checkpoints at the next midnight; a fight's damage and fame reach the country's history within 10 s; a formation standing in a loaded city appears as its own people and vehicles, one for one by id, and a vehicle destroyed in play is a wreck in the war world within 10 s |
| X7a | Hosting from the game: solo, and a friend joining | Yes | Two PCs on one host run the obstacle course together; a hero made on one host is refused on another |
| X8 | Several players in different cities (pulled forward: run as soon as X7a and Plan 1's streamer exist) | Yes | 4 players in 4 cities: server frame under 33 ms and memory under 12 GB; a fight 500 km from the centre passes the slice's checks. Sets the real player target |
| X9 | The paper on screen; the catch-up report | Yes | Each morning's paper on screen equals the rule file's `editionWords` for that country and day, for 3 mornings of war 1; a second player in the other country sees a different paper for the same morning; after a simulated week away, the report lists every change the back editions carried and nothing they did not |
| X10 | Leading a country: the war room | Yes; needs the trunk's part A, or a ruling to build it here first | A player takes Germany in war 1 and gives one order of each kind through the same doors the planner uses; an order the planner's door refuses is refused with the same words; no enemy position drawn on the map is newer than the fact in the country's book (checked every tick of one war); when the player leaves, the planner resumes on the next tick |
| X10b | The owner's ideas not in the browser game yet: leader personalities, law and budget levers, relations that move, falling leaders, office through fame | No for the rules, Yes to see them | Rules only: given the same book, a dominance-heavy and a safety-heavy leader choose different orders in at least 60% of 1,000 trials and neither ever gives an order the plain planner's doors refuse; no pair of countries leaves 1-5 or moves twice within 30 days in a 10-game-year run; every change of law, budget step or leader prints one news line with its cause; a hero over the thresholds wins a scripted election |
| X11 | Server settings | Yes | Each setting changes its number in a test |
| X7b | The screenless server | Yes; needs the engine built from source | A server with no screen runs for 24 hours with players joining and leaving; no crash, memory flat within 5% |
| X12 | Seasons | Yes | A season runs start to finish at a compressed clock and names a winner |

X0-X5 need no Unreal and share nothing with the controller, so they can run alongside it. They follow the trunk: when
the browser game's war layer changes (it is being built now), X0 is run again and every later stage is re-checked
against the new golden cases. The commit of the trunk the golden cases came from is written in `Data/manifest.json`.

## Needs a plan that does not exist yet

- **How friends find and join each other, and how a server knows who you are.** Robert, 2026-11-12: the game is sold on
  **Steam**. So Steam provides identity, the friends list, invites and "join game", and a hero on a server is tied to a
  Steam account. Unreal ships Steam connections for this, and they are in the 5.9 install (`OnlineSubsystemSteam`,
  `SteamSockets`). Publishing on Steam needs a Steamworks developer account, which is Robert's to open. The stage for this sits
  between hosting from the game (X7a) and the screenless server (X7b).
- **Hosting official season servers**: cost, where, who watches them.
- **A world map of the war for more than two countries**: places, roads and bases for every pair that can fight. The
  trunk has one hand-built theatre.

## Open questions for Robert

The trunk's own ten open calls (the ten numbered rows of `docs/design/WAR-LAYER-2.md` L196-207: who leads, how long a unit is without orders, clones, prisoners, paper names, canon
characters and the rest) are asked in the browser game's document with defaults, and are not asked again here. Unreal
ports whatever stands there.

1. The day: 180 real minutes (the war clock as built, tied to the one-eighth travel scale) or 120 (the time document)?
   The plan takes 180.
2. Office through fame, later: the thresholds, and whether force is allowed everywhere or only in some governments.
3. What a leader who is a player cannot do (for example, change the Treaty).
4. Season length and what winning means.
5. The living-world systems are built and switched off waiting on you. They work per person in a city, and would feed
   the strategy layer from below. On?
6. When a solo player quits, should the world stay frozen (the plan), or catch up the missed time on return?
7. The owner's ideas that are not in the browser game yet (leader personalities, law and budget levers): build them in
   the browser game first and port them, or build them in Unreal's rule files and offer them back? The plan takes the
   second, after the port.

Answered: Steam (2026-11-12). Whether operations happen between countries at peace (yes, in his own words, in the
trunk's document).

## Risks

- **Players spread across the world** multiply the server's work and collide with the re-centring fallback. X8 measures
  it before any player number is promised.
- **A strategy layer nobody can read is noise.** Every decision must print as one news line with its cause. The trunk
  already holds itself to this (every order names the report it answers; every story traces to an event); the port keeps
  it and it is in the pass lines for X4, X9 and X10b.
- **The port will not match to the last digit by itself.** JavaScript and C++ give the same answers for plain
  arithmetic but not always for functions like sine and logarithm, which the distance sums use. A difference in the
  last digit can flip a dice roll and from there a whole war. X2 finds out early; the fix is to port those few
  functions as fixed code shared by both, or to export the distances as data.
- **The trunk is still moving.** The war layer gained three parts in two days. Golden cases go stale; the manifest
  commit and the X0 re-run are the guard.
- **Part A and the government work are not merged.** X10 waits on them or on a ruling.
- **Player leaders can be griefers.** Server settings and limits on levers (question 3) are the guard.
- **Official seasons cost money to host.** They come last.
