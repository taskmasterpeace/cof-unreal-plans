# What each damage type leaves behind: looks, levels, materials, bodies

Written 2026-11-13 from Robert's words that day. Proposal; nothing built. It covers what a hit **leaves** on a vehicle,
a body, a wall or the ground, and how a body **reacts**. The flash of the hit itself and the power's own effect are the
effects plan's (`docs/effects/EFFECTS-PLAN.md` section 5: colour, size and element come from the power's data; fire adds
embers and heat shimmer, cold frost mist, electric arcs, toxic and acid drips). This file sits beside it and uses the
same element numbers.

Robert: "This is where we lean into Unreal... where Unreal shows what it could do." "All of this stuff got to react to
our materials." "It's different levels of the acid... different levels of the electricity. Fire is different levels."
"Again, all of this stuff has to be optimized."

## The nine damage types

The game's list (`_lsw-master/src/data/damage-types.js` L2), each resisted per character (`resistOf`, L19-30; magic is
resisted by Resolve).

| Type | Covers (Robert's words) | Level 1 (light) | Level 2 (heavy) | Level 3 (extreme) |
|---|---|---|---|---|
| **Physical** | fists, slams, throws, a baseball bat, crashes | scuffs, small dents, dust | deep dents, cracks, bent panels | crumpled, torn, broken through ("punch through") |
| **Ballistic** | bullets, shells, rounds | bullet holes, chipped paint, sparks on metal | clusters of holes, shattered glass, holed panels | shredded panels, a part shot off |
| **Energy** | blasts and beams, **in the power's own colour** | a scorched mark with a glowing edge in that colour | a melted, glowing crater; heat shimmer | a hole burned through, glowing rim, smoke |
| **Fire** | flame | soot and blistered paint | flames on the part, **heat shimmer** (Robert: "that visible... shimmering"), dark smoke | engulfed: the part burns until put out or gone, burnt-out shell after |
| **Cold** | frost and ice; **temperature is a thing in this game** | frost on the surface, breath and mist | ice crusting edges and joints, a slowed body | frozen solid: ice that cracks and shatters when hit |
| **Toxic** | **poison and gas** | a stain, a faint haze | a lingering gas cloud, sick colour on skin | a dense cloud that stays and drifts with the wind |
| **Acid** | levels of acid | etched spots, smoke wisps | running metal, holes eating in | a part dissolving away, a pool on the ground |
| **Magic** | spells, in the caster's colour | a faint glowing mark | runes or cracks of light | the surface warped or changed |
| **Electric** | "a whole bunch of different stuff": shocks, lightning, tasers | sparks, a scorch dot | arcs crawling over metal, lights flickering, a body locked stiff | a blackened strike, electronics dead, a ring of blast |

The level comes from how much damage the hit did against what the thing can take (its points, its armour), so the
same pistol round is level 1 on a tank and level 3 on a window.

## Reacting to materials

The game's own materials (`src/data/voxel-spec.js` L24-60: rock, concrete, wood, glass, steel, plaster, soil, brick,
plants, snow and the canyon and mountain rocks, each with its points and whether it holds metal) decide what each type
does:

| Material | Physical | Ballistic | Energy and fire | Cold | Acid | Electric |
|---|---|---|---|---|---|---|
| Wood, siding, roofs, plants | splinters | holes, splinters | **catches fire** and spreads | frosts | stains | scorches, can catch fire |
| Glass | cracks, then shatters | holes with cracks | cracks from heat, melts at level 3 | frosts over | etches | none |
| Steel, tin, chain link (metal) | dents, bends | sparks, holes | **glows red then white**, melts at level 3 | ice on it, can go brittle | runs and holes | **arcs travel along it** to anyone touching |
| Concrete, brick, rock | chips, cracks, chunks | chips and dust | scorched black, cracks from heat | frost | pits | scorch only |
| Plaster, stucco | caves in | big holes | blackens | frost | eats in | scorch |
| Soil, snow | craters | puffs | snow **melts to water and steam** | snow hardens | pits | scorch |

Water (Plan 19 section 6): fire goes out, cold freezes the surface, electric spreads through it to anyone in it.

## How a body reacts to being hit (Robert's picks for fliers, and the same for everyone)

Robert chose these (2026-11-13): small flinches by the direction of the hit; aim knocked off separately from the body,
with **Composure** making aim steadier; heavy hits drop you, **Toughness** deciding whether you drop and **Grit**
whether you recover; and **a chance to go full ragdoll**, for variety. The chance and the style follow the damage type:

| Type | What the body does |
|---|---|
| Physical | flinch by direction; heavy: knocked back the three ways the game has (planted, driven back, off the feet); the best chance of ragdoll |
| Ballistic | sharp small flinches per round, aim knocked off; heavy calibres can drop you |
| Energy | pushed along the beam or blast; a high ragdoll chance on big blasts |
| Fire | a flinch away from the heat, then patting or flailing while burning |
| Cold | slowed, stiff; a frozen body falls as one piece, not a ragdoll |
| Toxic | coughing, staggering, slowed; no knock-back |
| Acid | a recoil and clutching at the spot |
| Magic | per spell |
| Electric | **locked stiff and shaking**, then a drop: the body is rigid, not loose |

The ragdoll chance rises with the size of the hit against your Toughness, so a tough body rarely goes limp and a fragile
one often does. In the air, "drop" means knocked out of the sky (Plan 19, ideas for a flier being shot).

## How it stays cheap (Robert: "all of this stuff has to be optimized")

- **One master material for vehicles, one for props and buildings, one for bodies**, each reading a few numbers per zone
  and per damage type. A hundred burning cars cost little more than one.
- **Marks are small projected stickers**, a fixed number per object, oldest removed first.
- **Fire and gas are pooled effects** with a hard cap per area; past the cap the farthest one is dropped first. Burning
  and smoke beyond view distance stop simulating and keep only their look.
- **Heat shimmer** is one screen-space distortion near the camera, not one per flame.
- **Every number is on the quality dial** (R4): fewer marks, shorter smoke, no shimmer on low settings.
- **Saved with the cell**: what burned stays burnt, what melted stays melted (Plan 13). Smoke and gas clouds are not
  saved; they fade.

## Bigger things (Robert: "living super weapons versus anything")

Bigger planes, the mothership (320 long, two decks), giant walkers and the dinosaurs all use the same three looks per
model (intact, damaged, destroyed), the same damage zones and the same materials. Unreal draws large numbers of
polygons cheaply now, but the game keeps its faceted style: the gain goes into scale (more, bigger things on screen),
light and material, not into smoothing the shapes.
