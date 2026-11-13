# Plan 10: Powers and combat

Status: PLAN ONLY (2026-11-12). Nothing here is built. Comes after the controller (Plan 8). Corrected 2026-11-13 from
executed results (job J84; every change is listed in `R6-EXECUTED-CORRECTIONS.md`).

## The fact that makes this portable

All 385 roster powers run through ONE data-driven engine. A power is a row of numbers with a `type`; the engine has 48
type handlers, of which the roster uses 29, and no per-power code (`src/engine/abilities.js` L407-1576). So Unreal needs
the handlers built once, and the powers come across as data.

| Powers by engine type | Count |
|---|---|
| dash | 55 |
| buff | 47 |
| projectile | 45 |
| melee | 39 |
| cone | 38 |
| charge | 25 |
| beam | 24 |
| construct | 17 |
| rifle, rush | 15 each |
| volley | 13 |
| teleport | 10 |
| meteor | 9 |
| about 16 rarer types | the rest |

By family: biological 115, cosmic 109, elemental 84, tech 49, force 26, psionic 2. The wider library holds 526 powers
(the 385 plus creator powers, mounts and others). The 13 types in the table cover 352 of the 385. Each power has up
to about 130 optional numbers: cost, cooldown, damage,
range, radius, blast, speed, charge, damage type, knock, launch, lunge, homing.

The slice already has four of the big types in simple form: melee, projectile (ki blast), charge (energy ball), beam.

## How it is built in Unreal

- **The handlers go in the engine-free rule file first**, one type at a time, with tests against exported results from
  the browser game. Same method as the controller.
- **The ability system question.** Unreal's ability system gives cooldowns, costs, stacking buffs and (later) network
  play. But its effects are fixed assets that cannot be made from data during play (checked in the 5.9 source). The
  supported route is a small set of generic abilities, with each power's numbers passed in per use. That fits us
  exactly: one generic ability per engine type (29 for the roster, 48 in all), each fed a power row.
- **Damage before the full door exists.** The controller's contact, tackle and slam rules (Plan 8, rings 1, 5, 6) are
  tested on their raw numbers, before armour and resistances. The stat export (P0) and the damage door (P2) need no
  Unreal, so they are done during the controller phase and the same tests are re-run through the door when it lands.
- **Recommendation:** build the handlers in our own rule file now (needed for parity and tests either way), keep the
  ability system as an optional wrapper to adopt when buffs, many simultaneous effects or multiplayer make it worth its
  weight. Adopt Gameplay Tags now; they are cheap.
- **Looks come last and separately.** Every type gets one good-looking effect template (Niagara), tinted and scaled by
  the power's family and numbers. 29 templates dress 385 powers.

## Stats and what they drive (`src/data/ranks.js` L122)

| Stat | Drives |
|---|---|
| FGT | jab damage x(0.9 + 0.02 per point) |
| AGL | evade cooldown x(1 - 0.02 per point over 1); jump rank |
| MGT | knock-back, bullet toughness, lift. This is the sheet's strength. Bullet toughness starts at strength 6: a 20-point bullet does 20 to SARGE (strength 4), 16.6 to KANO (6), 9.8 to SOL (8) and 1.58 to TITAN (9, metal) (`Data/cases/damage_door_grid.json`, `Data/cases/damage_door_bodies.json`) |
| VIG | health: 70 + 9 per point (79 at 1, 115 at 5, 160 at 10; `Data/cases/power_stat_sheet.json`) |
| INT | cooldowns x(1 - 0.013 per point over 1). Confirmed, and the mood multiplies on top ("quick" x0.82, "certain" x0.92, "slow" x1.15): Solar Flare's 0.5 s is 0.4805 s on SOL's own sheet (Intelligence 4) and 0.3186 s with Intelligence 10, the Tactician talent and the "quick" mood (`Data/cases/power_ki_cooldown_rules.json`) |
| AWR | sight x(0.8 + 0.04 per point) |
| RES | recovery from stun, ki refill x(0.85 + 0.03 per point). Confirmed against a base of 8 ki a second standing and 3 while a power charges. Not in this plan before: holding guard brings the refill to 30 a second; each torso wound level cuts the refill 7%; being hurt gives back 0.4 ki per point of health lost, and a blocked hit gives none (`Data/cases/power_ki_cooldown_rules.json`). In a city, a guard held with no foe within 55 u turns into the power charge after 0.5 s, at 40 a second (`docs/proof/J11.md`; the two figures were not re-read for this correction) |
| PRE | charisma; no combat effect |

All run 1-10. Rank ladder: 1 to 4,999 over 15 bands. The threat scale has 10 rungs, Civilian to Cosmic.

## Damage and injury

- **Nine damage types**: physical, ballistic, energy, fire, cold, toxic, acid, magic, electric.
- **One door for all damage** (`entity.js` L983, where `takeDamage` starts). The real order is the table below. It is
  what 643 real hits show (`Data/cases/damage_door_grid.json`, `Data/cases/damage_door_bodies.json`), with the source
  read where no case tells two stages apart. Until 2026-11-13 this plan gave one line ("... resistance, nanite, guard, armour bar (55%),
  shield, phase, frozen x1.3, health"), which was wrong about the guard and about frozen.

| # | Stage | What the executed cases show (KANO, 115 health, hit by SOL for 20 unless said) |
|---|---|---|
| 1 | Attacker's mood | "certain" x1.12 (20 becomes 22.4); "hollow" x0.8 (16) |
| 2 | Predator, worn armour, pierce | No case, except worn armour (the SWAT set only) |
| 3 | Resistance by damage type | toxic x0.25 (5), acid x0.7 (14), magic by the body's Mental stat (19.2 on KANO) |
| 4 | Ballistic plate and bullet toughness | a bullet does 16.6 to KANO, 9.8 to SOL, 1.58 to TITAN |
| 5 | Two side effects that land even when the hit is stopped later | Magic drains ki: on a phased body a magic hit does 0 damage and still takes 15.36 ki. Acid corrodes for 5 s: on a phased body, and when a shield soaks the whole hit |
| 6 | Nanite panel | No case |
| 7 | **Guard, open-sky rules** | Paid for in ki, before the armour bar: 20 damage costs 20 ki (65 to 45) and 0 health |
| 8 | Armour bar | Takes 55% of what is left: on a bar of 30, a 20-point hit takes 11 off the bar and 9 goes on |
| 9 | Shield | Soaks what is left, point for point (a 14-point acid hit leaves a shield of 40 at 26) |
| 10 | Phase | Stops the hit (0 damage) |
| 11 | **Guard, city rules** | A multiplier applied after the armour bar, the shield and phase: x0.42 (20 becomes 8.4), x0.12 for a strike (2.4), x0.5 for a damage-over-time tick. Only from the front: the same hit from behind does the full 20 |
| 12 | Frozen | x1.3 only above a threshold: for a strike, or a hit whose knock-back is over 30. A plain 20-point energy hit on a frozen body does 20; a strike does 26; energy with knock-back 40 does 26, with knock-back 20 does 20 |
| 13 | Health, and ki back | 0.4 ki returns per point of health lost; a blocked hit returns none |
| 14 | Wound and stun checks | below |

  So there are **two guard rules**, not one: under open-sky rules the guard pays in ki before the armour bar; under city
  rules it multiplies after the armour bar, the shield and phase. The slice has a short version of this door (guard,
  wounds, stun, knockout). The game's own report lists 9 known mismatches in the full one
  (`docs/combat-states-and-damage-report.md` L105-113); the port fixes them or copies them on purpose, one by one, with
  Robert's word. That report was not matched against the cases one by one.
- **Wounds**: arm, leg, torso, three levels each. A wound opens on one hit of at least 16% of max health, counted after
  resistances and toughness: KANO (115 health) is wounded by a hit of 18.4 and not by one of 18.285. The same 19.55
  sent as acid or as a bullet lands as 13.7 and 16.2, under the line, and opens none. The three levels cap current
  condition by 5 / 12 / 25% (KANO: 109.25, 101.2, 86.25). Max health itself never changes
  (`Data/cases/damage_wounds_stun.json`). Serious ones can add hidden medical conditions (16 exist).
- **Stun**: 24% of max health inside 2 s. Confirmed: a hit of exactly 24% stuns and one of 23.9% does not; the stun
  lasts 1.7 s divided by recovery (1.6 s on KANO); a second stun inside 4 s is refused
  (`Data/cases/damage_wounds_stun.json`). **Status effects**: burn, bleed, frozen, shocked, blinded, grabbed and more
  (the survey listed 17; the count is unverified until the export).
- **The practice dummy has its own resistances, so tests against the dummy alone mislead.** The dummy takes toxic at
  x0.25, acid at x0.7 and magic at x1.1 (`Data/cases/power_type_cone.json`, `setup.target.resist`), so every toxic and
  acid number recorded on it is already after resistance. An ordinary flesh body has the same toxic and acid figures
  but its own magic figure (KANO x0.96, SOL x1.03), and a metal body is different again (toxic x0, acid x1.6, fire
  x0.6; `Data/cases/damage_resistances.json`). The dummy also never stuns, bleeds or takes wounds. Every power case
  was therefore run twice, on the dummy and on the live body KANO; a port has to match both.

## Combat beyond the slice

| Piece | Browser game | Plan |
|---|---|---|
| Strike / grab / guard triangle | Built | Port with the controller's contact and throwing rings (Plan 8, rings 1 and 6) |
| Parry, counter stance, back-grab, aimed throws, lift-off hold, tackle-to-pin, knife stabs | Built | Port after the triangle |
| Dodge, cover, hard lock | Built | Dodge is in Plan 8; cover and lock here |
| Weapons (29 rows in the armoury table; the survey counted 16 firearms and 7 blades among them) | Built | After powers; gun sound layering from the studio's gun-audio rules |
| Kicks | Clip lists only | New work; the purchased kickboxing and kung fu packs supply clips |
| Ground game, floor finishers, escapes, submissions | Designed, not wired | Stays a design item; the combat designer owns it |
| Directional damage | Robert called it a gap | Stays a design item |
| Enemy brains | 8 fighting styles picked from a character's powers; sight and sound memory | Port the style rules to the rule file; run them in Unreal's StateTree (in the install, off by default) or plain code if it gets in the way |
| Creatures | 83 rows, body-part attacks, no brain of their own | After enemies |
| The Psyche | 7 emotions, 20 personalities, mood changes damage, cooldowns, guard and speed | Port the mood multipliers with the damage door |

## Robert's rulings this plan follows

- Real time, never turn based.
- "You punch them or you blast them and they go flying back really far - that's one thing we're missing."
- Powers are what you ARE, gear is what you HOLD. Powers are separate from characters.
- Beams must curve as you turn ("a Dragon Ball Z beam is different").
- A new mechanic goes on a couple of different characters before it counts.

## Order

| # | Stage | Needs Unreal? | Done when |
|---|---|---|---|
| P0 | Export the 526 powers, stats, damage types, wounds, status effects (Plan 4) | No | Counts match: 526 / 385 / 9, and the status list counted |
| P1 | Export results from the browser game for up to 5 powers per engine type (damage dealt, ki spent, time on cooldown). Sixteen types have fewer than 5 powers, so about 98 cases | No | Each case replays the same twice. Done by job J11: 98 powers, each fired at the practice dummy and at the live body KANO (`Data/cases/power_type_*.json`), plus 643 damage-door hits |
| P2 | Port the full damage door and the stat formulas to the rule file | No | Every case's damage matches within 1% |
| P3 | Port the 13 biggest engine types (covering 352 of the 385 powers) | No | Their cases match within 1% |
| P4 | Those 13 on screen with plain effects; a power bar to pick any of the 352 | Yes | 20 powers picked at random each fire and hit a dummy for the exported damage within 1%, at 30, 60, 120 and uncapped frames per second |
| P5 | The remaining 16 types | Both | All 385 fire; a script fires each once and logs zero failures |
| P6 | Effect templates, one per type | Yes | 29 templates exist; each holds the frame under 16.7 ms with 20 on screen; Robert marks each keep or redo |
| P7 | Enemy brains: the 8 styles | Yes | Each style defeats a standing dummy within 60 s; a log shows each style using its own power choice at least 80% of the time |
| P8 | Weapons, kicks, the rest of melee | Yes | Each of the 29 weapons fires or swings and deals its table damage within 1% |

## Open questions for Robert

1. The 9 known mismatches in the damage order: fix them in the port, or copy the game exactly first and fix later?
2. Is the ground game (floor fighting, submissions) wanted before or after powers?
