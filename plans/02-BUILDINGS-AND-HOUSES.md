# Plan 2: Buildings and the house formula

Status: PLAN ONLY (2026-11-12). Nothing here is built. Corrected 2026-11-13 from executed results (job J84; every
change is listed in `R6-EXECUTED-CORRECTIONS.md`).

## What the browser game does

- **Every building is made of small blocks.** Walls use 1 u (19 cm) cells, columns and terrain 4 u cells. No building is a
  model file. [`src/data/voxel-spec.js`]
- **58 materials**, each with hit points and a "power gate" (the weakest attack that marks it at all): glass 15 / 0,
  wood 60 / 0, brick 180 / 35, concrete 1200 / 250, steel 2200 / 500. [`voxel-spec.js` L22-85]
- **391 catalogue entries in 20 kinds**: 303 authored (houses 1-3 floors, apartments 4-8, towers 10-30, shops, offices,
  civic) and 88 generated houses. [`src/data/city-recipe-catalog.js`, counted in Node on the trunk, 2026-11-13]
  (This line was "119 types" until 2026-11-13, from `docs/BUILDING_TYPES.md`, which still says so. The 88 are
  new: houses from the house generator in 7 wealth tiers, 22 seeds, each facing 4 ways, `city-recipe-catalog.js` L41-53.
  New cities put these on their house lots. The export must count the catalogue, not the document.)
- **Interiors are real.** A vendored generator (EveryDoor, 9 archetypes) plus our own floor stacking. Only ONE interior can
  be loaded at a time. [`src/engine/voxel/README.md`]
- **Houses come from a formula**, not a list: bedrooms, baths, storeys, garage, floor area, culture, wealth and climate in,
  a checked floor plan out. [`src/engine/voxel/house-generator.js`, `house-plan.js`]

## The house formula (what must be proven in Unreal)

| Rule | Number | cm |
|---|---|---|
| Wall thickness | 1 u | 19 |
| Storey / ceiling, garden flat | 17 / 16 u | 323 / 304 |
| Storey / ceiling, ranch | 19 / 18 u | 361 / 342 |
| Storey / ceiling, family house | 20 / 19 u | 380 / 361 |
| Storey / ceiling, mansion | 24 / 23 u (foyer 47 u) | 456 / 437 (893) |
| Front door, most classes | 12 x 13 u | 228 x 247 |
| Front door, family house | 12 x 14 u | 228 x 266 |
| Front door, mansion | 14 x 16 u | 266 x 304 |
| **Interior doors** | down to 8 u and 10 u wide (`house-plan.js` L168, L254-256) | **152** and 190 |
| Window | 8 u wide, sill 4 u | 152, sill 76 |
| Corridor upstairs / ranch hall | 8 / 10 u clear | 152 / 190 |
| Stairs | 1 u rise, 2 u run | 19 rise, 38 run |
| Stair lane | 10 u main, **8 u basement**, 16 u mansion | 190, **152**, 304 |
| Stair headroom | well opens 16 treads below the top | (13 treads stopped the hero dead) |
| Furniture | 1 u off walls; door keep-out 8-9 u deep; walking lines clear | |

Checks the browser game already runs on every plan (`checkHousePlan`, `house-plan.js` L866-919): no overlaps, doors on shared
walls, no door onto a stair, windows only outside, every room reachable from the front door, kitchen + living + bath exist,
every bedroom has a closet and a bed, kitchen beside dining, every lived-in room has a window.

**Those checks are on paper. They do not prove a body fits.** The Unreal proof is physical:

1. Port `house-plan.js` + `house-generator.js` + the checker to engine-free C++ (`COFHouse`). Tests compare against the
   browser game for a grid of seeds: 4 classes x 6 climates x 16 culture choices (no culture, or a code 0 to 14) x 5
   seeds = 1,920 houses. That is the real grid, exported by job J12: 480 plans in each of
   `Data/golden/houses_apartment.json`, `Data/golden/houses_ranch.json`, `Data/golden/houses_family.json` and
   `Data/golden/houses_mansion.json`, and none fails the game's own checker. (This line said "13 regions" and 1,560 until 2026-11-13.) Culture changes the
   railings, the door leaves and the stairwell openings cut round the railings. It does not change the room layout:
   rooms, doors, windows, stairs and walls are the same with and without a culture in all 1,800 cultured plans.
   No Unreal needed.
2. In Unreal, build each test house as plain boxes and send the hero through it on autopilot: in the front door, into
   every room, up every stair, out again. It uses the real collision body and the real camera.
3. Pass means: no room unreachable, no stuck spot, the head never hits a slab on a stair, the camera never ends up inside
   a wall or outside the house, and the hero can turn around in every corridor.
4. Run it three times: the standard body, a short body and a large body, because the game has all three.

Two things this walk needs that do not exist yet: an autopilot that can find its way from room to room (it reads the
plan's door list and walks door to door), and a ladder move for the family house's attic (Plan 5).

### Fit numbers to check first (paper maths, to confirm in engine)

| Thing | Game | Unreal Manny default | Note |
|---|---|---|---|
| Hero height | 182 cm | about 180 cm capsule | fine |
| Body radius | 2.2 u = 42 cm | 34 cm (checked in the 5.9 source) | **Use 42 cm.** The doors were tuned for it. Decided here so B3 is not blocked; Robert can overrule |
| Step-up | 2.5 u = 47.5 cm | 45 cm (checked) | set to 47.5 so the game's 19 cm treads and kerbs behave the same |
| Stair slope | 1 in 2 = 26.6 degrees | walkable up to 44.8 (checked) | fine |
| Narrowest door 152 cm vs body 84 cm wide | 1.8 body widths, 34 cm spare each side | | fine on foot, tight in a fight |
| Basement stair 152 cm | same | | fine |
| Chase camera | game: 485 cm behind. Slice: 248 cm up on the ground, 122 cm in the air | | **longer than most rooms**. Needs an indoor camera rule |
| Ceiling 304 cm vs flight | hover needs about 60 cm clearance | | flying indoors is possible but tight |

The camera is the real risk, not the doors. The browser game fades the outer shell and roof and keeps the camera under the
ceiling [`citylook/camera.js` L70, note 014]. Plan: an indoor camera mode (pull in to about 200 cm, over the shoulder),
switched by "am I under a roof", proven in the course (Plan 5, station 9).

No rule for flying indoors exists in the game today. That is a design question for Robert, listed below.

## Real sizes against the game's sizes (Robert asked, 2026-11-12)

| Thing | Real world | The game today |
|---|---|---|
| A person | 5'9" is 175 cm | The hero is 182 cm (6'0") |
| Ceiling | 8 ft (244 cm) is standard; 9 ft (274 cm) in newer houses | 10 ft (304 cm) in a flat, up to 11.8 ft (361 cm) in a family house, 14.3 ft in a mansion |
| Floor to floor | about 9 to 10 ft | 10.6 ft (323 cm) and up |
| Front door | 3 ft wide x 6'8" tall (91 x 203 cm) | 7.5 ft wide x 8.1 ft tall (228 x 247 cm) |
| Inside door | 2'6" to 3 ft wide (76 to 91 cm) | 5 ft (152 cm) at the narrowest |
| Hallway | 3 to 4 ft (91 to 122 cm) | 5 ft (152 cm) |
| Stair step | 7.5 in up, 10 in along (19 x 25 cm) | 19 cm up, 38 cm along |
| Roof ridge, one-storey house | about 15 to 18 ft | follows from the storey height |

The game's rooms are bigger than real on purpose: Robert asked for higher ceilings and spaces (quoted in
`house-plan.js`), because a camera behind the hero needs the room. Most third-person games inflate interiors by a fifth
to a third for the same reason. The doors, at two and a half times real width, are the outlier.

With an indoor camera in Unreal, the houses can move closer to real size. The course's door and corridor gauge
(Plan 5, station 10) is how the number is found: it finds the smallest door, hallway and ceiling the hero and camera work
in, and the formula is then set a safe margin above that. Because every size is a number in the house formula, changing
them is a data change, not a rebuild.

## How to draw block buildings in Unreal

Three ways were weighed:

| Way | Good | Bad |
|---|---|---|
| A. Keep blocks as the truth, build meshes from them at run time | Same saves, same damage rules, any hole anywhere, all 391 catalogue entries (20 kinds) work day one | We write the mesher. Run-time meshes cannot use Nanite (the Nanite builder is an editor-side module in the 5.9 source) |
| B. Modular kit pieces + Chaos fracture | Looks best out of the box, Nanite | Throws away the block rule ("any block can be removed"), saves do not carry over, all 20 kinds (391 entries) need kits |
| C. Blocks as truth, drawn as kit pieces until first damaged, then swapped to a block mesh | Nanite and instancing for the 99% of buildings nobody has hit yet, blocks where it matters | Two looks must match; the swap must not pop |

**Recommendation: A first, C as the upgrade.** A gets every building on screen with the real rules. C is how the far and
untouched city gets cheap. B is rejected: it breaks Robert's 2026-09-20 ruling (Minecraft-style block removal) and the saves.

## Stages

| # | Stage | Needs Unreal? | Done when |
|---|---|---|---|
| B0 | Export golden houses and 12 golden buildings (blocks + plans) from the browser game as JSON | No | Done by job J12: 1,920 house plans (in four class files, plus 200 plans made from household numbers) and 12 building files in `Data/golden/`, each checked by the browser game's own checker |
| B1 | Port the block store, material table and house formula to engine-free C++ with tests | No | 1,920 houses match; checker agrees with the browser game on every one |
| B2 | Mesher: blocks to mesh on worker threads (merge flat runs), with collision | No for the maths, Yes to see it | A 3-floor house meshes in under 50 ms; triangle count within 2x of the browser game |
| B2b | Autopilot house routing: walk door to door from the plan's door list | No for the route maths | Route visits every room on paper for all 1,920 plans |
| B3 | One house on screen in Unreal, walkable, with the indoor camera | Yes | Autopilot house walk passes |
| B4 | The 1,920-house walk, overnight | Yes | Pass list + a list of every failing seed |
| B5 | All 391 catalogue entries (20 kinds: 303 authored + 88 generated houses) as shells in a city cell; interiors load for EVERY building in the near cells, not one | Yes | A scripted pass loads all 391 entries without error, and a scripted walk enters one building of each of the 20 kinds |
| B6 | Furniture (the game's 295 contents cards) as instanced meshes; signs and shopfronts | Yes | A furnished house and a shop |
| B7 | Way C: kit pieces for untouched buildings | Yes | Same city, frame time at least 20% lower; before and after pictures of a swapped building differ by under 1% of pixels |

B0-B2 can start today with Unreal closed.

## Open questions for Robert (none block B0-B2)

1. **Flying indoors.** Allowed everywhere, only in big rooms (mansion foyer, malls, warehouses), or never under a ceiling
   lower than some height?
2. **Body radius.** The plan uses the game's 42 cm. Say so if you want it slimmer.

## Risks

- The mesher is new code and the most likely place for hitches when a wall breaks. It gets its own tests and a time budget.
- EveryDoor is vendored JavaScript (MIT). Porting it is real work; the house formula is ours and comes first, the other
  eight archetypes follow.
- Finding a compact house layout takes 0.5-1 s per house in the browser. In C++ on worker threads this should fall well
  under the time it takes to walk up to a door, but it is unmeasured.
