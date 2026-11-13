# Plan 6: Speed, the needs gate, and space

Status: PLAN ONLY (2026-11-12). Nothing here is built. Corrected 2026-11-13 from executed results (job J84; every
change is listed in `R6-EXECUTED-CORRECTIONS.md`).

## What actually gates speed in the browser game today

Robert's picture: the needs system is the gatekeeper for the high speeds. What the code does is close to that, but through
three separate gates, and only one of them is a need in the hunger-and-thirst sense.

| Gate | How it works | Where |
|---|---|---|
| **1. Ki (energy)** | Boost gears cost ki every second: 2.6 / 6 / 12 for gears I / II / III. Empty ki drops you out of the gears. Hunger (under 15% food) cuts ki regen to x0.7, thirst to x0.6, and at zero the cut doubles. So a hungry, thirsty hero cannot hold top gear for long | `data/movement-gears.js` L10-18, `data/needs.js` L32, L41-57, `entity.js` L1420, L1922 |
| **2. Breath** | Holding top gear in thick air builds strain: (speed / 200 u/s) squared x air density. WINDED at 0.25, GASPING at 0.6, BLACKOUT at 1.0 (a 1 s stagger). After 1.2 s of gasping, top gear locks out until strain is under 0.35. These figures were confirmed by a scripted 10-minute flight (`Data/golden/flight_scripted_10min.json`), which also showed one thing this plan did not say: after a blackout the held gear key does nothing until it is let go and pressed again. Thinner air up high means less strain. Masks, an air-shield power, stamina, rank and an alien body resist it (up to 95%). Machines and sealed suits are immune | `data/air-pressure.js`, `core/movement-gears.js` L58, L116-148 |
| **3. Altitude** | A hard cap of 1,678 mph inside the air. It lifts above 12 km, the top of the atmosphere | `data/flight-cap.js`, `flight-bands.js` |
| Space | A suit gate (PRESSURE, HARD or DEEP by world) and suit air for 8, 10 or 24 game hours | `COMBAT_MANUAL.md` section 60, `data/suit-air.js` |

The needs themselves: Food drains in 36 game hours, Water 24, Sleep 18, and for machines Maintenance 72 and Energy 30.
One game hour is 2.5 real minutes. Effects start under 15%. Nothing kills you. Sleep does not touch speed at all today.

One direct link exists: for machine characters, low Maintenance cuts move speed to x0.9 (`needs.js` L48).

**So the gatekeeper is half built.** For living heroes, hunger and thirst reach speed only through ki regen. If the needs are meant to be THE
gate, the port is the moment to make that link direct. Proposal for Robert to rule on:

- Each boost gear needs a minimum body state to engage, not only ki. Example: gear II needs no need in the red;
  gear III needs food, water and sleep all above 35%; space-speed bands need above 50% and a sealed suit or a power.
- The HUD says why a gear is refused, in plain words ("too hungry for full boost").
- Breath stays as the in-the-moment limit; needs become the day-to-day limit.

Until he rules, the Unreal port copies today's behaviour exactly.

## The speed ladder to carry over

| Rung | Speed | Notes |
|---|---|---|
| Walk / run | Unreal slice: 32.4 u/s (6.2 m/s), which is the browser game's 30 u/s default x 1.08. Browser game as run: a plain walk comes out at speed x 0.972 at 60 steps a second, not x1.08 (32.08 u/s for a body of speed 33; 29.16 for the default 30), `Data/cases/ground_speed_entity.json` | ground gears x1.3 / 1.7 / 2.2 of speed x 1.08 (confirmed for the flyer profile in the same file) |
| Flight base | 38.9 u/s | in the slice. Browser game as run, SOL (speed 34): 47.8 u/s under city rules, 52.9 under open-sky rules (`Data/cases/flight_vertical_entity.json`) |
| Boost I / II / III | x1.5 / x3 / x5 | in the slice: 58.4 / 116.7 / 194.5 u/s. Browser game as run, SOL: 71.8 / 143.5 / 239.2 u/s under city rules and 79.3 / 158.6 / 264.4 under open-sky rules. The same three multipliers on a faster base |
| Bands 1-6 (design doc) | 31, 43, 55, 134, 559, 1,678 mph | band 6 is the in-air cap |
| Supersonic | 3,803 mph | above the atmosphere |
| Hypersonic | 17,448 mph | orbit needs this |
| Escape | 25,054 mph | leaving Earth |
| Interplanetary | 671,081 mph | the Moon is a trip at this rung |
| Lightspeed, Warp, Intergalactic, Instant | design only | |

Two things the survey found that need settling before the port:

- Gear III has three different figures: 1,322 u/s (562 mph) in `flight-cap.js`, 264 u/s in an older table, and
  194.5 u/s in the Unreal slice (38.9 x 5). Run for real on 2026-11-13, a flyer (SOL) holds 239 u/s at gear III under
  city rules and 264 u/s under open-sky rules (`Data/cases/flight_vertical_entity.json`). So the "older table" figure is
  what the game does in open sky today, 1,322 u/s is a cap and not a gear speed, and the slice's 194.5 is slower than
  both. The slice keeps 194.5 inside cities until stage S0 decides.
- Walk speed: the slice walks at 32.4 u/s (the game's 30 x 1.08). The browser game's own plain walk, as run, is
  speed x 0.972 at 60 steps a second (`Data/cases/ground_speed_entity.json`; Plan 8), so the slice walks 10% faster
  than the browser game. The slice's figure stays.
- The 1,678 mph cap "binds nobody today" by the game's own comment, because no hero reaches it.
- The file that would tie speed to rank was planned but does not exist, so rank does not gate speed today.

## Space: what is built, what is only designed

| Piece | State today |
|---|---|
| Sky darkens as you climb, no cut (the exact heights come from a design note and were not found in code) | Built, but it only changes the sky |
| Low-orbit depart: past the ceiling with the burner lit opens a world map, then a cinematic | Built, in modes that are parked |
| Space layer: flybys, the heliopause at 123 AU, powers-of-ten out to nearby stars | Built as a cinematic, not flown |
| Earth globe with real day and night | Built |
| Moon, Ganymede | A switch of environment, not a flight |
| Continuous Earth-to-Moon flight | Not built. The space-trip report says so plainly |
| Warp travel | Not built |

Robert's rulings: the game never teleports you. Leaving Earth should feel like No Man's Sky. He wants to see a 1:1 Earth
from orbit, with travel "almost a blur". Faster than light is for space.

## The hard problem, stated plainly

The ground is a flat map at 1:8. Orbit is supposed to show a round Earth at 1:1. Those cannot be the same object. The
browser game hides the join with a cut. Unreal lets us hide it without one, but it has to be designed.

**Plan: three nested spaces, blended by altitude, never a load screen.**

| Space | From - to | What you are flying over | Scale |
|---|---|---|---|
| Ground | 0 - 12 km | the generated city and country (Plan 1) | 1 u = 19 cm, country 1:8 |
| High air | 12 - 100 km | the same ground fading into a globe that is curved to match and textured from the same map, so the city you left is under you | blend |
| Near space | 100 km - Moon | a real round Earth and Moon at true size and distance | 1:1 |
| Deep space | beyond | planets and stars drawn at a shrunk scale around the hero, who stays near the centre | scaled |

Unreal 5.9's world limit is about 44 million km from the centre (checked in the engine source: `UE_LARGE_WORLD_MAX`,
`EngineDefines.h` L41). So Earth-to-Moon (384,400 km) fits at true size. Mars at its closest (about 55 million km) and
everything beyond do not, which is why deep space is drawn at a shrunk scale with the hero near the centre.
UNVERIFIED: how physics and effects behave hundreds of thousands of km out. Spike S5 measures it.

Coming back down is the same blend in reverse: the globe under you becomes the map of the place you are over, and the
generator builds the city as you drop through 12 km.

## Stages

| # | Stage | Needs Unreal? | Done when |
|---|---|---|---|
| S0 | Settle the numbers: the three gear III figures (as run: 239 u/s in a city, 264 in open sky, against the slice's 194.5), walk (slice 32.4; browser game speed x 0.972); write the full speed ladder as one table in the tuning file | No | One table, each row with its source; Robert signs off the top rungs |
| S1 | Export a 10-minute scripted flight from the browser game (speed, ki, strain, needs every 0.1 s). Port ki, the five needs, breath strain and the altitude cap to the engine-free rule file, plus a test switch to set any need | No | The port's log matches the export within 1% at every sample |
| S2 | Course station 16: 5 km straight and a climb to 12 km | Yes | Top gear locks after 1.2 s gasping and returns under 0.35 strain; with Food at 10%, boost III lasts the predicted time within 5% |
| S3 | Altitude bands (same job as Plan 1's sky stage, done once): sky, haze, cloud base at 1.9 km, ground detail fading | Yes | Each band changes within 50 m of its height in `flight-bands.js` |
| S4 | Spike: the ground-to-globe blend between 12 and 100 km over one city | Yes | No load screen; no frame over 33 ms; a marked landmark stays within 1% of screen width of its spot through the blend |
| S5 | Spike: fly to 400,000 km from the origin and punch a dummy there | Yes | The slice's 12 autopilot checks pass out there |
| S6 | Earth and Moon at 1:1; fly between them at the Interplanetary rung | Yes | One unbroken flight in about 21 minutes at that rung, or faster by Robert's ruling on "almost a blur"; no load screen |
| S7 | Suit gate and suit air | Yes | No suit, no space; air runs down |
| S8 | Deep space at shrunk scale; the existing powers-of-ten trip, now flown | Yes | Reach the heliopause |

S0 and S1 can start today with Unreal closed. S4 and S5 are the two experiments that decide whether the rest is easy or
hard, and they should be done early.

## Open questions for Robert (none block S0-S1)

1. **Needs as the gate.** Keep today's indirect link (hunger starves ki), or make it direct (a gear refuses to engage when
   a need is too low)? The proposal above is the direct version.
2. **Sleep.** Today it does not affect speed. Should it?
3. **Rank.** Should the faster rungs also need a rank, as the missing speed-scale file intended?
