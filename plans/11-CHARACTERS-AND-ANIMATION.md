# Plan 11: Characters and animation

Status: PLAN ONLY (2026-11-12). Nothing here is built. The animation half runs alongside every ring of the controller
(Plan 8). Stages that show something on screen are checked at 30, 60, 120 and uncapped frames per second.

## What exists in the browser game

| Thing | Facts | Where |
|---|---|---|
| The blocky hero | one model, 53 bones, 46 runtime groups, 4,126 triangles, 6 materials, 12 built-in clips. 152 editable source pieces | `public/models/modular-hero/`, `docs/CHARACTER_AUTHORING_HANDOFF.md` L46 |
| Costume | 36 slots; parts switch on and off; 5 colour channels (suit, accent, dark, metal, skin); 16 colour fields, 20 region colours, camo, up to 8 tattoo layers | `src/engine/modular-costume.js` L37-55, L132 |
| Builds | hero, heavy (1.38 wide), agile (0.83), machine (1.20); size 0.85-1.25, muscle 0.8-1.3 | `modular-costume.js` L17 |
| Size ladder | mouse 0.044 to titan 3.6 | `src/data/size-change.js` L24-33 |
| Hero recipe | name, look, voice, stats, flight tier, guard type, evade, melee tiers, gifts, up to 3 talents, 2 gadgets, 6 power slots | `src/data/creator.js` L311-325 |
| Roster | 66 characters, all on the same body | `src/data/characters.js` |
| Civilians | seeded: 8 archetypes x 4 age bands x 2 sexes x 8 wealth tiers x 15 culture codes | `src/data/civilian-generator.js` |
| Creatures | 83 species on 28 body plans, with procedural rigs (four legs, hoppers, flyers, eight legs, swimmers), not the hero skeleton | `src/data/creature-catalog.js`, `src/engine/creatures/` |
| Clips | about 624 entries in 16 banks: controls 241, warworld 79, flight 60, zombie 45, grapple 27, parkour 24, blade 22 and more | `public/models/modular-hero/*motion-bank.json` |

## How animation works today, and why Unreal is a big step

There is no animation graph in the browser game. Clips are picked in code by speed and state (idle under 2, walk over 2,
sprint over 25), with a short leg cross-fade. Eight-way movement sits behind a switch that is off. The move kits file is
not wired in. Ragdoll, foot locking and leg placement are all home-made.

Known problems the game's own notes list: a sprint pop patched in code, legs that snapped up to 122 degrees between pose
sources, feet sliding 26-42 cm during knock-back, and two rigs running at once under the body.

Unreal has all of this built in: blend graphs, layered upper body, motion matching, physical hit reactions, ragdoll with
get-up, foot and hand placement, and moving clips between skeletons. The slice does none of it yet (it switches whole
clips), which is the single biggest gap between how the slice plays and how it looks.

## Two decisions

**1. Which body the game is built on.**

| Option | Good | Bad |
|---|---|---|
| A. Our blocky hero, on its own 53-bone skeleton | It is the game's look. 624 clips already fit it with no conversion. Costumes, builds and the creator carry over | Unreal's purchased packs (flight, boxing, kung fu, knockdowns) must be converted onto it. One-time work, automated |
| B. Unreal's Manny, as in the slice | Packs fit directly | Not our look; our 624 clips and all costume work would need converting the other way |

**Recommendation: A.** Manny stays as the stand-in until the hero is imported, and every pack clip is converted onto our
skeleton with Unreal's retargeter (the slice already did 34 clips this way in the other direction). Epic's 5.9 notes
describe a new retarget step that keeps hands in place relative to bodies, which would matter for grabs and throws; the
critic could not find it in the install, so treat it as UNVERIFIED.

**This decision is needed at the start of the controller phase**, because the animation work is built on whichever body
is chosen. The hero import (stage A1) therefore sits in phase 1 of the roadmap.

**2. How costumes work in Unreal.** The hero is one skeleton with parts shown or hidden and tinted. In Unreal: one
skeletal mesh per part group, all following one lead skeleton, with colours as material parameters. A crowd cannot
afford 46 parts each (whether the 80 simulated people of the 2026-11-13 default or the 2,000 that Plan 12 only measures), so far-away people are merged into one mesh per recipe when created. The install has two tools for
that to try: the engine's skeletal mesh merge, and Mutable (Beta).

## Order

| # | Stage | Needs Unreal? | Done when |
|---|---|---|---|
| A0 | The hero is already exported to FBX (`SourceArt/Hero`); check it has all 46 groups and 53 bones. Export the 16 clip banks and the recipes for the 66 roster characters (Plan 4) | No (Blender) | 46 groups, 53 bones, about 624 clips counted in the exports |
| A1 | Hero in Unreal with part switching and colours from a recipe | Yes | All 66 roster characters build with zero missing parts; for each, the list of visible parts and the 5 channel colours equal the browser game's export |
| A2 | Convert the pack clips onto our skeleton (flight 60 first, then boxing, knockdowns) | Yes | Each converted clip plays with feet within 3 cm of the ground and hands within 5 cm of the source |
| A3 | Animation graph for the controller: 8-way locomotion with blending, flight poses by speed and tier, layered upper body | Yes | No foot slide over 5 cm at any walk speed; measured, not eyeballed |
| A4 | Hit reactions and ragdoll with get-up, replacing the home-made ones | Yes | Knock-back slide shows under 5 cm of foot skate (the browser game shows 26-42) |
| A5 | Ledge, ladder, hang and carry: hands and feet placed on the real surface | Yes | Hands within 3 cm of the ledge on the course's grapnel wall |
| A6 | Builds and sizes: heavy, agile, machine; child to giant | Yes | The house walk (Plan 2) passes with each |
| A7 | Civilians from the seeded generator, merged for crowds | Yes | 200 different civilians on screen with the frame under 16.7 ms |
| A8 | Creatures: one body plan at a time, starting with four legs | Yes | For each of the 28 body plans: walks 20 m with foot slide under 5 cm and lands each of its listed attacks on a dummy |
| A9 | Faces: the 7 expressions | Yes | Seven close-up pictures; shown unlabelled, a second reviewer names at least 6 of 7 correctly |

## Open questions for Robert

1. Build on our blocky hero (the recommendation) and keep Manny only as a stand-in: yes?
2. Children: the civilian generator has no child age band today. In or out?
