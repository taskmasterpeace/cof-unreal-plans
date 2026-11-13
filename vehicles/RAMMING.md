# Ramming an aircraft or a ship: the two laws, worked

Written 2026-11-13 for Plan 19; corrected the same day after the critic's read. Robert: "imagine a game where you could
literally fly and ram a plane if you wanted to. You might take damage, you might get knocked back, you might destroy
both of y'all."

**Update, later 2026-11-13: read `FLYING-CONTACT.md` first.** After Robert asked for momentum, guarding and grabs, the
better base turned out to be the browser game's own strength contest (`data/contact.js`), which already weighs momentum,
bracing and being caught out. It decides **who gives way** (bounce, struggle, pinned). The damage side below (what
breaks, who is hurt, no caps) still applies once the verdict is in.

The table below is printed by `ram-cases.mjs` in this folder (`node docs/vehicles/ram-cases.mjs` from the repo root).
**Law A's columns come from the browser game's own code, called directly** (the ram function and the hull's force gate);
Law B's are the proposal. Nothing here is built in Unreal. 1 game unit (u) = 19 cm; 194.5 u/s is the slice's boost III
(133 km/h).

Strength is the game's rank ladder (`data/scale.js` `rankOf`): an ordinary person (strength 5) is rank 25; the lift
ladder reads the same number (`data/contact.js` L29: "STR = min(100, rankOf(def))"). Rank 30 lifts 500 kg, 40 lifts
5 t, 50 lifts 50 t, 60 lifts 250 t (L30). Every hero in the cases has the stock 90 kg body (`data/body-mass.js` L12);
real roster bodies differ.

## Law A: the browser game's law, as it is today

`_lsw-master/src/engine/shared-impact.js` L90-126 (`resolveFighterVehicleImpact`), Robert's contract of 2026-09-16
(comment L57-89).

- Contact at 30 u/s or less is free (L91).
- **Who owns the hit** is whoever brought more of the closing speed: the craft's own speed on the contact axis against
  the hero's (L93-94).
- Craft-driven (it runs into you): you take `(closing - 30)^2 x 0.012`, at least 1, at most **60**, knock-back
  `closing x 1.35`, lift `closing x 0.5`, credited to the craft's pilot (L100, L109-111); the craft takes a token
  `closing x 0.22` at armour-piercing 0.8 (L112).
- Hero-driven (you ram it): the craft takes `closing x 0.55 x (your weight / 180 lb, held between 0.5 and 3)` (L55,
  L114). **On a fleet vehicle's hull** (every vehicle and AI aircraft the game spawns as fleet, `engine/game.js` L2791)
  it is **multiplied by your strength** through the force gate: the cube root of how many ordinary people's lift you
  have, 1 to 12 (`data/force-gate.js` L15-22, L30-36, applied in `engine/vehicle-combat.js` L66). **Airliner parts take
  the raw number** (`engine/airliner/airliner.js` L759-765, no gate), and airfield aircraft count strength as 1
  (`engine/aircraft-combat.js` L228). You take
  `(closing - 45)^2 x 0.010`, at most **40**, and a small bounce (L115-118). Your own ram is free below 45 u/s.
- **Armour**: the force must beat the craft's threshold first; aircraft have none, ships 60 (`force-gate.js` L46).
- One hurt per hero and craft in any 0.45 s (L98).
- A helicopter's rotor is a blade, not a floor: flying into the disc costs you `8 + speed x 0.28` (at most 34) and
  the craft `speed x 0.3` (at least 10) (L130-156).
- Hull points: jet-a 140, helicopter 170, ship 2,400 (`data/vehicle-mounts.js` L50-54); a parked airfield aircraft 160
  (`engine/aircraft-combat.js` L79). Airliner parts: wing panel 420, wing box 600, engine 300, fin 260, nose 180, skin
  90 (`data/airliner-recipes.js` L171, L365). The cases use jet-a's 140.

What Law A cannot do:
1. **The craft's mass never enters, and your strength counts only on fleet hulls.** A 250 t airliner and a 1.2 t car hit you the same at the same speed.
2. **The faster side owns the whole hit.** Head-on with a jet faster than you, your strength stops counting: any hero
   deals the token 109 and the jet flies on with 31 of its 140 points.
3. **The caps (60 and 40) mean nobody ever dies from one plane hit.** An ordinary person hovering in a jet's path takes
   60 at most.
4. There is no "punch through": the hero always bounces off.

## Law B: the proposed law (slice choice, needs Robert's yes)

Same damage door (so armour and resistances still apply), same free 30 u/s, same one-hurt window, same rotor rule.
Three changes, for aircraft and ships only:

1. **Effective mass.** In flight your mass is your body plus your lift (the ladder above) when you are **braced** (a
   committed ram: flying, fists or shoulder first), and just your body when you are not (thrown, tumbling, stunned).
   This replaces Law A's strength multiplier, so strength is not counted twice.
2. **Break or bounce.** The force on the part you hit is `0.55 x closing x (effective mass / 82 kg, at least 0.5)`,
   less the part's armour. If that beats the part's points, the part breaks and **you go through**, keeping
   `sqrt(1 - points / force)` of the closing speed; you are hurt only by the speed you lost, and the craft is pushed by
   the same momentum (tiny for anything big). If not, **you bounce**: the two bodies trade momentum by their masses (the
   impulse the game already uses between two bodies, `shared-impact.js` L34, factor 1.08) and you are hurt by your own
   change of speed.
3. **No cap on your hurt.** Hurt = `(your change of speed - 30)^2 x 0.012`, Law A's craft-driven curve without its cap,
   then through armour and resistances. Toughness (the game's impact tolerance, `shared-impact.js` L7-10) is a tuning
   dial to be set from the cases.

Either side can die. The craft dies by the part it loses, through the browser game's airworthiness rule
(`engine/airliner/airliner.js` L270-283: a twin flies on one engine; under 62% lift on either wing, no tail, or nobody
at the controls means a glide down). The hero dies by bouncing off something too strong.

## The cases (Law A's receiver is named in its column; the browser game has no ships, so those Law A cells are what a fleet hull would give. Masses are slice choices: fighter 12 t, helicopter 5 t, 777 250 t, ferry 2,000 t, cargo ship 50,000 t; the cargo ship's 5,000-point plate is a slice choice too)

| Case | Hero | Craft | Closing speed | Law A: hero hurt / craft hurt | Law B effective mass | Law B force vs part | Law B result | Law B: hero's speed lost, hurt | Law B: craft pushed |
|---|---|---|---|---|---|---|---|---|---|
| Ordinary-strength flier, boost III, head-on | rank 25 | fighter jet 12 t at 300 | 495 u/s (338 km/h) | 60 / 109 of 140 (fleet hull) | 391 kg | 1297 vs jet-a hull 140 | through, keeps 94% | 27 u/s, hurt 0 | 0.9 u/s |
| Strong hero, boost III, head-on | rank 30 | fighter jet 12 t at 300 | 495 u/s (338 km/h) | 60 / 109 of 140 (fleet hull) | 590 kg | 1957 vs jet-a hull 140 | through, keeps 96% | 18 u/s, hurt 0 | 0.9 u/s |
| Strong hero, tumbling | rank 30, not braced | fighter jet 12 t at 300 | 495 u/s (338 km/h) | 60 / 109 of 140 (fleet hull) | 90 kg | 299 vs jet-a hull 140 | through, keeps 73% | 134 u/s, hurt 130 | 1.0 u/s |
| Jet overtakes a hovering ordinary person | rank 25, not braced | fighter jet 12 t at 300 | 300 u/s (205 km/h) | 60 / 66 of 140 (fleet hull) | 90 kg | 181 vs jet-a hull 140 | through, keeps 48% | 157 u/s, hurt 194 | 1.2 u/s |
| Hero catches up with a jet from behind | rank 30 | fighter jet 12 t at -150 (same way) | 45 u/s (30 km/h) | 0 / 31 of 140 (fleet hull) | 590 kg | 176 vs jet-a hull 140 | through, keeps 45% | 24 u/s, hurt 0 | 1.2 u/s |
| Strong hero into an airliner wing tip | rank 30 | 777 250 t at 150 | 345 u/s (236 km/h) | 40 / 209 of 420 (airliner part, no strength) | 590 kg | 1363 vs wing panel 420 | through, keeps 83% | 58 u/s, hurt 9 | 0.1 u/s |
| Ordinary-strength flier, not braced, into a wing root | rank 25, not braced | 777 250 t at 150 | 250 u/s (171 km/h) | 60 / 55 of 600 (airliner part, no strength) | 90 kg | 151 vs wing box 600 | bounces | 270 u/s, hurt 691 | 0.1 u/s |
| Heavy hero into an airliner nose | rank 60 | 777 250 t at 150 | 345 u/s (236 km/h) | 40 / 209 of 180 (airliner part, no strength) | 250090 kg | 577876 vs nose 180 | through, keeps 100% | 0 u/s, hurt 0 | 0.1 u/s |
| Hero rams a helicopter | rank 30 | helicopter 5 t at 80 | 275 u/s (188 km/h) | 40 / 189 of 170 (fleet hull) | 590 kg | 1086 vs helicopter hull 170 | through, keeps 92% | 22 u/s, hurt 0 | 2.6 u/s |
| Hero into a cargo ship side | rank 30 | cargo ship 50,000 t at 30 | 225 u/s (154 km/h) | (no such craft; as if a fleet hull: 40 / 154) | 590 kg | 828 vs hull plate (slice choice) 5000 | bounces | 242 u/s, hurt 542 | 0.0 u/s |
| Hero into a cargo ship side | rank 40 | cargo ship 50,000 t at 30 | 225 u/s (154 km/h) | (no such craft; as if a fleet hull: 40 / 226) | 5090 kg | 7604 vs hull plate (slice choice) 5000 | through, keeps 59% | 93 u/s, hurt 48 | 0.0 u/s |
| Hero rams a moving ferry | rank 40 | ferry 2,000 t at 30 | 225 u/s (154 km/h) | (no such craft; as if a fleet hull: 40 / 226) | 5090 kg | 7604 vs ship hull (vehicle-mounts.js L54) 2400 | through, keeps 83% | 39 u/s, hurt 1 | 0.1 u/s |

What the cases say, in plain words:
- **Any braced flier goes through a fighter jet** and barely slows. A tumbling one still goes through but is hurt (130).
  An ordinary person hovering in a jet's path wrecks the jet and dies (194). Law A gives all three the same 60 and 109.
- **One ram through an airliner's outer wing does not bring it down.** The tip panel carries 14% of that wing's lift
  and the next 20% (`airliner.js` L52), so losing one leaves 80% or more, above the 62% line. It takes the root panels,
  the wing box, both engines or the tail.
- **An unbraced body bounces off an airliner's wing root** and takes 691: deadly for most. Ramming big planes carelessly
  has a price.
- **A cargo ship's side holds against anyone below about rank 39** (rank 38 makes 4,837 against 5,000; rank 39 makes
  6,070). The browser game's 2,400-point ship hull gives way to rank 40.
- **The craft hardly moves** in every case (under 3 u/s). Stopping a plane with your hands is a catch (a grab, Plan 8),
  not a ram; it is not in this law.
- These are first numbers for the tuning stage; Plan 19's pass lines test the rule, not these values.

## Robert's rulings (2026-11-13)

1. **One law for everything**: Law B for every vehicle, cars and bikes included. Law A stays only as recorded browser
   cases.
2. **A hero can die from ramming a plane.** No caps.
3. **A held guard counts as braced**, so guarding against a ramming plane brings your lift into the hit; punching a
   plane is a strike through the same door.

In plain words, the one law: when two things meet, the game asks how heavy each is, how fast they were closing, and
how strong and braced you are. Strong and braced makes you count as heavy, so a plane breaks on you. Loose or weak
makes you light, so you get thrown. Whatever breaks, breaks; whoever is hurt, is hurt; nothing is capped.
