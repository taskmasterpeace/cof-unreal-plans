# Plan 1: The world

Status: PLAN ONLY (2026-11-12). Nothing here is built. No Unreal needed to read or refine it.
Source facts: survey of `D:\dev3\lsw` (file:line in brackets) and `docs/UE59-FIELD-GUIDE.md`.

## What we are carrying over

| Fact from the browser game | Number | Where |
|---|---|---|
| Unit | 1 u = 19 cm. Hero 9.6 u, storey 17 u | `src/data/world-scale.js` L25-30 |
| World Cell | 1344 u = 255.36 m a side. Sector = 16 x 16 cells (4.09 km) | `world-scale.js` L34-37 |
| Country shrink | City-to-city distance = real / 8 (Robert 2026-09-25) | `world-scale.js` L45 |
| Cities | 1,109 real cities in 198 countries (198 nations in the country table). The last 62 cities and 30 countries were typed by hand on the trunk and are marked as estimates there | `src/data/cities.js`, `countries.js` |
| City sizes | village 3 cells a side, town 6, city 12, metro 24 and up, hard cap 48 (12.3 km). The side grows with population inside the class: a generated metro is 24 x 24 cells (`Data/golden/city_metro.json`), but Miami (6.08 million people) is 25 x 25 = 625 cells (`Data/golden/city_miami.json`) | `world-scale.js` L91-99 |
| Whole world | 287,560 cells, 13.2 M building records | The browser game's WORLD1 report. It is not in the trunk export, so these two figures were not re-checked |
| Generator | Seeded, repeatable, no random calls. Zones ring out from downtown | `src/data/city-generator.js` |
| Cell grid | Four planes that never overlap (planet, country, procedural, stand-alone cities). Cell ids are in the save format | `world-scale.js` L69-74 |
| Projection | Sinusoidal equal-area, +z south | `src/data/world-address.js` L102-107 |
| Streaming today | 3 cells around the player real, 3x3 ring coarse, rest "far city". At most 5 real buildings | `cell-stream-plan.js`, `city-streaming.js` |
| People today | 80 pedestrians (128 until 2026-11-07, when Robert asked for fewer), 28 cars + 2 buses, 22 live characters | `src/data/city-density.js` |

The browser limits (5 real buildings, 250 draw calls) are the reason to move. (The 80 pedestrians are Robert's choice since
2026-11-07, not a limit.) The rules above are not limits;
they are the design, and they move across unchanged.

## The main decision: the world is generated, not authored

Unreal's World Partition and HLOD are built for maps a person lays out in the editor. Our world is 287,560 cells made from
a seed. Nobody will place them by hand. So:

1. **The generator is the truth, same as the controller.** Port `city-generator.js`, `world-district.js` and
   `country-grid.js` into an engine-free C++ file set (`COFWorld`), the way the hero's rules live in `COFCore.h`.
   Same seeds in, same city out. Tested without Unreal against "golden" outputs exported from the browser game.
2. **Our own cell streamer.** A small C++ system asks "which cells are near the camera", calls the generator for them on
   worker threads, and spawns one actor per cell. This is a port of `cell-stream-plan.js`, with bigger numbers.
3. **Unreal does the drawing.** Instanced meshes for repeated pieces, Nanite for static kit pieces, Lumen for light.
4. **PCG is a helper, not the backbone.** Use it for dressing a cell (street furniture, trees, clutter) where a graph is
   quicker than code. PCG can generate at run time in 5.9 (checked in the engine source). How well that holds at flight
   speed is untested; that is spike W2 below.
5. **World Partition only for hand-made places** (a hero base, a set-piece map, the obstacle course).

Why not bake all 287,560 cells into Unreal maps: it would be tens of gigabytes, every generator change would mean a
re-bake, and saves point at cell ids that must keep matching the browser game.

## Scale and coordinates

- 1 u = 19 cm exactly, so a cell is 25,536 cm. Do not round to Unreal's usual 256 m.
- Unreal uses 64-bit positions, and 5.9's world limit is about 44 million km from the centre (checked in the engine
  source). A city, its country and a whole planet at 1:8 all fit.
  UNVERIFIED: physics and effects far from the origin. Spike W3 measures it; the fallback is to re-centre the world on the
  player per city, which the browser game already does past 131,072 u.
  (Trunk note, 2026-11-13: the trunk now streams cells out to 1,048,575 u, 199 km, from the anchor city, and moves the
  anchor to the nearest city when the game is saved and resumed: `cell-stream-plan.js` L35, `world-cell/reanchor.js`.)
- Keep the four cell planes and the cell id format, so a save names the same place in both engines.
- Flat map per region with the sinusoidal projection, as today. A true round globe is a space-plan question (Plan 6).

## What gets better in Unreal (targets to prove, not promises)

| Thing | Browser today | Unreal target for the first city | Proof |
|---|---|---|---|
| Real buildings near the hero | 5 | every building in the 3x3 cells | frame time while flying a lap |
| Pedestrians | 80 | Follow the game: about 80 individually simulated people near the hero (the trunk's world people), real actors within reach. 2,000+ Mass entities is only measured, not targeted, until Robert rules (note below, 2026-11-13) | frame time + count on screen |
| Traffic | 30 | The game's 30; how many more fit (300+ was the guess) is only measured; a rammed car becomes a physics car (same rule as today) | ram test |
| View distance | skyline to 3 km | skyline to the city edge (12 km metro) | screenshot from 2 km up |
| Shader freeze | 37-39 s seen in Miami | none in a cooked build | hitch log |

**Crowd note, 2026-11-13. NEEDS ROBERT'S RULING.** The trunk dropped pedestrians from 128 to 80 on Robert's word of
2026-11-07 ("I think crowds should be less") and added world people: fewer people, each one simulated
(`src/data/world-people.js`; Plan 12 has the summary). The default is to follow the game: a small number of individually simulated people near the player (the trunk's world people), with the 2,000-figure crowd kept only as a measurement of what Unreal can afford, not a design target, until Robert rules.

## No pop-in (Robert, 2026-11-12: "I want to avoid pop in")

His reference is the real world: far things sit behind a blue haze, and the haze thins as they get closer. "The web thing
did it all wrong." This is a rule for every stage that draws the world, with a number:

- **Haze is physical.** Unreal's sky and fog model the air itself, so distance turns things blue-grey the way it does
  outside. It is set from real numbers (visibility in km by weather), not painted on.
- **Nothing ever appears in clear air.** Anything new (a cell, a building, a person, a car) must first appear either
  smaller than 4 pixels on screen or behind enough haze that it differs from the sky by under 5%, and fade in over at
  least half a second. Detail then grows as it nears.
- **Detail changes without a visible switch.** Static pieces use Unreal's Nanite, which changes detail continuously.
  Block buildings made during play cannot use it, so they cross-fade between their far and near forms.
- **Measured, not judged.** A recorded flight is checked frame against frame: any object whose screen size or brightness
  jumps by more than a set amount between two frames is logged as a pop. The target for a lap is zero.
- Robert's photo of real haze, when he takes it, becomes the reference picture the sky is matched to.

## Stages (each ends with a proof, in order)

| # | Stage | Needs Unreal? | Done when |
|---|---|---|---|
| W0 | **Export the data.** Node script writes cities, countries, coordinates, borders, generator outputs for 6 test cities (one per class + Miami + a coastal one) as JSON | No | Files exist, counts match (1,109 / 198) |
| W1 | **Port the generator** to engine-free C++ with tests against the 6 golden cities | No | Same cells, zones, roads, lots for all 6 |
| W2 | **One cell on screen**: grey-box blocks, roads, lots from W1 data. Spike PCG runtime vs plain C++ spawning | Yes | One cell renders; build time per cell measured |
| W3 | **Streamer**: fly across a 12-cell city at boost III with no hitch over 33 ms. Far-origin test at 1,000 km | Yes | Autopilot lap passes at 30/60/120/uncapped fps |
| W4 | **Far city**: shells, boxes, skyline tiers (port `far-tiers.js` distances, then widen) | Yes | From the city edge at 2 km up, every tower over 10 floors is drawn; frame under 16.7 ms |
| W5 | **Life**: crowds and traffic. Owned by Plan 12 (stages L2-L5); listed here only for order | Yes | See Plan 12 |
| W6 | **Country**: highways, hamlets, open terrain between cities at 1:8 | Yes | Fly Miami to Orlando: no load screen, no frame over 33 ms |
| W7 | **Sky**: time of day, 14 weather states. Altitude bands are done once, as Plan 6 stage S3 | Yes | All 14 weather states switch on command; a full day cycle runs |

W0 and W1 can start today with Unreal closed.

## Open questions for Robert (none block W0-W1)

1. **Travel time.** Miami to Orlando is about 42 minutes at 1:8; your yardstick was 20. Keep 1:8, or change the ratio or the
   cruise speed? (`world-scale.js` L46-53 flags this as still open.)
2. **How big the crowd is. NEEDS YOUR RULING (rewritten 2026-11-13).** This plan guessed 2,000 people and 300 cars. On
   2026-11-07 you said "I think crowds should be less"; the trunk went from 128 pedestrians to 80 and added world people.
   The default until you rule: follow the game: a small number of individually simulated people near the player (the trunk's world people), with the 2,000-figure crowd kept only as a measurement of what Unreal can afford, not a design target, until Robert rules.

## Risks

- Mass is still marked Experimental in 5.9 and Mass Traffic is not in the engine (it ships in Epic's City Sample).
  Fallback: instanced pedestrians with our own simple walker, as the browser game does.
- 5.9 is a Preview. Anything saved in it will not open in 5.8. The generator and data stay engine-free for that reason.
- One editor at a time on SOVEREIGN (16 GB). World work and controller testing cannot run at once.
