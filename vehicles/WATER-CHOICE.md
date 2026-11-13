# Water: Epic's free Water plugin or Dynamic Real Water?

Checked 2026-11-13, read only (nothing bought, signed in or downloaded). Robert ruled for Epic's free plugin and asked
what Dynamic Real Water would save and what we would lose. Sources: the
[Fab listing](https://www.fab.com/listings/84495b74-94b4-45d1-a08d-2feac2ecb92f), the seller's
[docs](https://naturallevel.dev/assets/dynamic-real-water/introduction/),
[roadmap](https://naturallevel.dev/assets/dynamic-real-water/roadmap/),
[multiplayer page](https://naturallevel.dev/assets/dynamic-real-water/concepts/multiplayer-model/),
[underwater guide](https://naturallevel.dev/assets/dynamic-real-water/guides/underwater/), the
[Epic forum thread](https://forums.unrealengine.com/t/natural-level-dynamic-real-water-multiplayer-ready-fft-water-simulation-for-ue5/2531924)
and an [80.lv article](https://80.lv/articles/gpu-accelerated-fft-water-system-for-unreal-engine-5) (4 June 2025).
"Not seen" means the seller does not say.

## What it is

$99.99 Personal / $199.99 Professional, C++ with full source, Unreal 5.4-5.8 (no 5.9 plan seen), 4.7 of 5 from 13
ratings, one developer (last post about an update: 23 June 2026). It **replaces** Epic's Water plugin rather than
building on it. The listing bans AI training ("Allows usage with AI: No"); whether our AI seats may read its code is
not stated, so the seller would be asked before buying.

## What it does for us

| Need | Dynamic Real Water | Epic's free Water (in 5.9, Experimental) |
|---|---|---|
| Waves | GPU wave simulation driven by wind, up to 8 layers, swell, presets | Simpler waves, the same on every PC |
| Deep ocean look | Below the surface by itself: light fades and darkens with depth, caustics, sun shafts, up to 8 underwater lights. **Deepest it handles: not seen** | An underwater screen effect only; the deep look is ours |
| **Water pressure by depth** | **Not seen.** It gives only the surface's height, slope and speed | **Ours either way.** The browser game's crush rule comes across |
| Seabed | Uses our ground; does not make one | Same |
| Swimming and diving | **Not built** ("no swimming controller", item 7 on its roadmap) | Unreal's basic swimming |
| Boats float | **Strong**: each shape floats by its material (32 real densities), a steel hull floats while it holds air, ragdolls float | Basic floats (sample points) |
| Boat engines, sails, wakes | **Strong**: engines, jets, sails, oars, five hull types, hulls that rise at speed, V wakes, two ready boats | None; ours |
| Damage, flooding, sinking, ramming | **Not built** (item 9 on its roadmap) | None; ours |
| Multiplayer | **Strong**: the server owns the wave clock and works out all floating; boats sent 30 times a second; seen within 1 km; late joiners catch up; a dedicated server skips the visuals | Same waves everywhere; boat networking is ours |
| Round planet | Not seen; the far ocean is flat to the horizon | Flat, in a bounded area |
| Rivers, lakes | Yes (shapes cannot change during play) | Yes |
| Cost and future | Paid, one developer, we would update it to 5.9 ourselves | Free, Epic carries it to 5.9 and beyond |

## What it would save

The surface: good waves, boat physics and the networking for anything that floats. Roughly a few weeks of work on
stage F10 (boats).

## What it would not save

Everything that makes our water ours: deep diving, pressure by depth, the seabed, swimming, sinking and ramming ships,
and a round planet. Two of those sit low on its roadmap; pressure and the planet are not on it at all.

## What we would lose

Epic's path forward (we would carry it to 5.9 ourselves), independence from one developer, and a flat far ocean that
does not fit a curved planet. Also $100-200.

## Verdict

**Stay on Epic's free Water plugin (Robert's ruling).** Revisit Dynamic Real Water only if surface ship fights become
a core feature, and only when it runs on 5.9 and the seller says AI tools may read its code. Either way, pressure,
diving, sinking and the planet's ocean are built by us, in the rule file, from the browser game's rules:

- Space and deep water are **separate hazards** in the browser game: vacuum and crush (`data/environments.js` L16-34).
  An energy body answers vacuum fully and crush not at all; a metal body answers some crush (L211-237).
- Divers have crush depths by gear: 32 m breath-hold, 66 m air tank, 260 m rebreather (`data/scuba-gear.js` L34-69);
  past the limit, damage rises with depth (`engine/world-cell/dive.js` L11).
- So a hero who can fly in space can fly underwater, but only as deep as their crush answer allows.
