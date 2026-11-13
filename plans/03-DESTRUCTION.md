# Plan 3: Destruction

Status: PLAN ONLY (2026-11-12). Nothing here is built. Corrected 2026-11-13 from executed results (job J84; every
change is listed in `R6-EXECUTED-CORRECTIONS.md`).

## The rules we already have (they move across unchanged)

**One damage rule** [`src/engine/voxel/store.js` L52-61]: if the attack's power is under the material's gate, nothing
happens. Otherwise the block loses hit points, and at zero it is gone. One hit reaches a ball of blocks, at most 128 u wide.

**How hard each attack hits structures** [`src/engine/city-session/damage.js` L22-27]:

| Attack | Multiplier | Hole size |
|---|---|---|
| Punch | The strength ladder, not (STR / 5) squared. On the trunk a punch carries the striker's strength-ladder power (`src/data/force-gate.js`: 1 at strength 5 and under, 1.26 at 6, 2.44 at 7, 4.20 at 8, 5.81 at 9, 6.74 at 10), divided by the punch's weight factor. That gives x1 at strength 5, x3.34 at strength 8 and x4.73 at strength 10 (`Data/golden/block_damage.json`, cases `wielder-power`, `punch-str8:brick-wall`, `punch-str10:brick-wall`). The older (STR / 5) squared, at least 1 (x2.56 at 8, x4 at 10) still runs, but only when a hit carries no ladder value (cases `punch-strength-only-*`) | power / 120, between 1 and 10 u |
| Blast | 1 to 6 by power level | same |
| Beam, bullet | 1 | as wide as the beam |
| Explosion breach | Both formulas are real and do different jobs (settled by job J12). An explosion uses x8 (`blastBreach` in `data/blast.js`): amount = damage x 8 and gate = 60 x power to the 1.3, applied as two spheres, the second at 0.3 of the amount, 0.7 of the gate and 1.5 times the radius (`engine/blast.js` L44-45). Example: damage 40, power 3 gives amount 960 and gate 250.27, then 288 and 175.19 at radius 13.5. The x6 (`construct-kit.js` L417) is the construct hammer's separate hit on the surface it lands on. `Data/golden/block_damage.json`, cases `blast-breach`, `breach-source-*`, `breach-r30-p3-d40:brick-wall` | 1.5 to 9 blocks |
| Thrown body | hurts the body above 30 u/s (`entity.js` L1438-1444); "a hard enough launch breaks through the wall instead" (`force-feel.js` L126). The exact wall rule is to be read out in D0 | |

Worked examples, as run (`Data/golden/block_damage.json`). A strength-5 punch with no buff, wound or momentum does not break brick: fully charged it is
34 at the wall, brick's gate is 35, and the hit is blocked (the amount was built by the exporter without the power buff, arm wound and momentum the game's swing multiplies in, `melee.js` L787; with about x1.03 or more of those it would reach the gate, not run). It marks wood and breaks one glass block. A strength-8 punch
(143) marks brick without breaking it. A strength-10 punch (229) breaks 14 brick blocks and only marks a steel section.
A power-4 blast (269) marks concrete and does not break it. (Until 2026-11-13 this line repeated the game's comments:
"a street punch breaks only brick" and "a STR 10 power punch (194) cuts steel sections". The executed results do not
agree. 194 is the older formula's figure, and that punch breaks 10 brick blocks.)

**Collapse** [`src/engine/world-cell/structural-collapse.js`, 2026-09-23]: the building is read in storey bands. A band
fails on any one of five conditions (this plan listed four until 2026-11-13):

| # | Condition | Cause the game gives | Seen in the executed cases? |
|---|---|---|---|
| 1 | Under 40% of its load-bearing blocks stand | walls | Yes: fails at 39.98%, stands at 40.03% |
| 2 | Two sides are under 35% | side | Yes |
| 3 | One side is under 35% AND under 60% of all its walls stand (the fifth condition) | side | Yes: with the west wall gone the band fails when the rest falls to 59.97% |
| 4 | Its floor plate is under 40% | slab | Yes: fails at 39.92% |
| 5 | Under 30% of its outer ring stands | outer ring | **Never reached.** In all 13 cases the walls rule or a side rule tripped first |

Taking out one whole wall alone never fails the storey. Executed: `Data/golden/collapse.json` (13 cases, each either
side of the flip). Everything above a failed band is removed and rubble mounds appear on the floor below.

**Fire** [`src/data/world-fire.js`]: wood, siding, shingle and foliage burn at 10-16 hp/s; 32 fires, 18 s each, 7 spread
generations. Brick, concrete and steel never light.

**Power vs. a house** [`src/data/power-scale.js` L47-57]: breach radius 4.5 u at rank 1 up to 60 u at rank 10; a house goes
from "scratch" to "LEVELLED" (95% of blocks gone). Robert 2026-09-24: strength 100 should completely destroy a house.

**Saving**: a journal of block changes per city, plus one "collapsed" rule per building. Repair is supported.

## What changes in Unreal

The rules stay. What Unreal adds is everything the browser could not afford:

| Layer | Browser | Unreal plan |
|---|---|---|
| Truth | block store | the same block store, ported to engine-free C++ (`COFBlocks`), tested against the browser game |
| Drawing a hole | re-mesh the chunk in a web worker | re-mesh on worker threads (Plan 2 mesher), swap in on the next frame |
| Falling upper floors | deleted instantly, no physics | the removed blocks become a handful of big physics chunks that fall and settle (Chaos), then turn into the same rubble mounds |
| Debris | 128 fragments, 11 per hit, 2.9 s | thousands of Niagara pieces, with the 20-50 largest as real physics bodies that can hit things |
| Dust, smoke, fire | 16 dust slots, 32 fires | Niagara, one always-on system fed by events (no first-use hitch) |
| Light | none | blasts, beams and fires as real lights (MegaLights) |
| Set pieces | none | a tower falling can be recorded once and replayed cheaply (Chaos Cache, Experimental) |
| Save | browser storage with a quota | a save file. Same journal format |

The upper floors falling as physics chunks is cosmetic first. What is gone and what stands is always decided by the block
rule, so the result is the same on every PC and in every save.

## Two rulings that conflict (needs Robert)

- 2026-09-20: "Minecraft-style block removal... Destroying a column must not automatically destroy or drop the roof."
- 2026-09-23 (#144): "what loses its support falls." The code follows this one.
- A third, approved but unbuilt direction: collapse must not auto-kill people; only a few chunks of debris do damage.

**This plan follows the later ruling** (support matters, by storey band), with the third as the damage rule for falling
chunks. If the first ruling still stands for some things (small props, single columns), say which.

## Gaps in today's rules that the port should settle

| Gap | Today | Proposal |
|---|---|---|
| Damage type | recorded, changes nothing (fire does not burn wood faster in the store) | a small table: type x material multiplier. Fire x wood 2, cold x glass 2, everything else 1. Robert rules on it |
| Metal flag | stored, unread | leave for the magnetism powers |
| Strength vs material ladder | PROPOSED doc, nothing reads it. (No longer true, 2026-11-13: the trunk has `src/data/force-gate.js`, one rule of force against a threshold, read by the structure damage door, rams and vehicle armour; note 179) | port the force gate with the damage rules in D0; keep the material gates |
| Water | breaking rock under the water line does not flood | out of scope for the first pass |

## Stages

| # | Stage | Needs Unreal? | Done when |
|---|---|---|---|
| D0 | Export golden cases from the browser game: hits (each attack kind x each target) and collapses, with the blocks before and after. Done by job J12: 120 hits and 13 collapse cases, in `Data/golden/block_damage.json` and `Data/golden/collapse.json`; the breach rule is read out above. Still to do: read out the thrown-body wall rule | No | 133 cases in two files; replaying each in the browser game gives the same result twice |
| D1 | Port the block store damage rule, strength multipliers, collapse bands and fire to engine-free C++ with tests | No | All 120 hit cases and 13 collapse cases match |
| D2 | A test wall of each material in the obstacle course; punch, blast, beam and thrown body each leave the right hole | Yes | Hole sizes match D1 within one block |
| D3 | Re-mesh time: a 10 u hole in a tower chunk in under 8 ms off the game thread, no visible hitch | Yes | Hitch log |
| D4 | Collapse: fail a band, upper floors fall as chunks, rubble mounds match the rule | Yes | Same standing blocks as D1, on video |
| D5 | Debris that matters: the big chunks can hit and knock back characters (ties to the body-collision rule) | Yes | A chunk knocks a dummy down |
| D6 | Fire, smoke, dust, light | Yes | A wooden house burns; a brick one does not |
| D7 | Save and reload a damaged city; repair | Yes | Damage is the same after restart |
| D8 | "Level a house": strength 100 leaves under 5% of blocks | Yes | Count + video |

D0 and D1 can start today with Unreal closed.

## Risks

- Physics chunks from a 30-floor tower can be thousands of bodies. Cap by size: only the largest N are physical, the rest
  are effects. The cap is a number to tune on SOVEREIGN.
- Chaos Destruction is Beta and Chaos Cache is Experimental in 5.9. The plan only uses plain physics bodies for chunks,
  which is production code. Fracture assets are an optional look upgrade later.
- Re-meshing during a beam sweep (a hole every frame) is the stress case. It is test D3's hard version.
