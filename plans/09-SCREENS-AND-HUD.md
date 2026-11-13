# Plan 9: Screens and the on-screen display

Status: PLAN ONLY (2026-11-12). Nothing here is built. Comes after the controller (Plan 8).

## What exists in the browser game

Over 30 player screens, all built as web page elements laid over the 3D view. Only the radar, the world map and the two
globes draw their own pictures. Paths are under `D:\dev3\lsw\src\engine`.

| Group | Screens | Size today |
|---|---|---|
| Front | title and cold open, character select with a live 3D figure, hero creator ("ORIGIN") | about 2,700 lines |
| In play | combat display (health, ki, guard, crosshair, lock, combo, hit numbers, radar, gears, compass), needs chips, wanted row, damage doll, interaction prompts, speech balloons | `hud.js` alone is 2,705 lines plus 1,157 of styles |
| Menus | pause, player menu (save, load, controls), options, how to play, inventory (5 sections), powers and loadout, Case File and career | about 3,000 lines |
| World | world map, city atlas, the two globes | about 2,100 lines |
| Devices | phone and laptop with about 25 apps, news (KMK 9) | about 3,000 lines |
| Shops and talk | shop counter, car shop, clinic, dialogue wheel, transcript | about 1,300 lines |

Data that carries over as it is: `hud-components.json` (33 display pieces, several still marked placeholder or proposed),
`ui-themes.json` (9 themes), about 676 power icons, 521 item pictures, the 121-row key table. Stat tiles exist too
(the survey said 24; unverified).

Every stage below that shows something on screen is checked at 30, 60, 120 and uncapped frames per second.

## The look: one palette, decided

The browser game has four palettes that disagree (the main token file, the "HUD kit" navy, a design doc's cobalt, and
COF Tactics). Robert, 10-18: "we need the black." The main display does not use it yet.

**Unreal starts from COF Tactics and nothing else:** black `#141204`, yellow `#FFBE0B`, red `#E71D36`, green `#20BF55`,
and blue; comic display type with halftone blue panels. The blue has two values in circulation: `#2457D6` in the game's
own stylesheet (`src/beta/beta.css` L3) and `#2854DB` in the look guide. Stage U1 picks one. Robert's own fonts (Heroes Legend, PG Roof Runners,
Badaboom BB, Helenski) go in if he supplies the files; until then the stand-ins.

Other rulings that carry over: icons are only for powers, everything else uses models (note 138); panels are
standardised; flags are never a letter pair; no purple anywhere (one character excepted); damage numbers show in the chase
camera with an off switch.

## How it is built in Unreal

- **Menus:** CommonUI, the engine's own menu framework. It handles gamepad focus, a stack of screens, and button pictures
  that switch between keyboard, Xbox and PlayStation (checked in the 5.9 source; see research note 1). The slice's
  on-screen bars are drawn the quick debugging way and get replaced.
- **One style sheet.** Colours, fonts, panel shapes and spacing live in one data asset. Every screen reads it. A screen
  never names a colour itself. This is what stops four palettes happening again.
- **The display reads the game, never the other way.** Health, ki, gears, needs and wounds are exposed once by the hero;
  each display piece listens. The 33-piece list becomes 33 small widgets, built in the order the controller needs them.
- **Words in a table from day one.** The browser game has its words written into code, with no languages (only the
  separate boxing build has English and Spanish). Unreal's text tables make translation possible later at no cost now.
- **Controls:** Enhanced Input, with rebinding built in (the browser game only offers fixed schemes).

## Order

| # | Stage | Why then | Done when |
|---|---|---|---|
| U0 | Export the display piece list, themes, key table and icon manifests (part of Plan 4's export) | No Unreal | Files exist with the counts above (33, 9, 121, 676) |
| U1 | The style sheet and a sample panel in the COF Tactics look | Everything reads it | One picture of the sample panel that Robert approves |
| U2 | Controller display: health, ki, guard, gears, crosshair, lock, hit numbers, wound doll, needs chips, prompts | The controller is first, and the course needs to show its state | Each of the 10 pieces shows the right value in a scripted test (set health to 37, read 37) |
| U3 | Pause, options, controls with rebinding, how to play | Needed to hand anyone a build | Every control reachable by gamepad alone; every button tapped once in a recorded pass |
| U4 | Title, character select, hero creator | Needs the blocky hero imported (Plan 11) | Make a hero with a pad only, start to finish |
| U5 | Inventory, powers and loadout | Needs powers (Plan 10) | Equip, swap and use an item and a power |
| U6 | World map, atlas, globes | Needs the world (Plan 1) | Pick a city on the map and arrive there |
| U7 | Phone and laptop, news, shops, dialogue | Needs people and the economy | One purchase, one conversation, one news item |

## Known gaps to fix on the way, from the browser game's own notes

- "Title screen is overwhelming"; the Case File lacks categories; onboarding does not explain everything.
- Display pieces marked proposed or placeholder: ammo, flight meter, signal bars, missions.
- Rings for actions like choke-out and cuffing are specified but not wired.
- THE SIGNAL has no screen yet in the main game; it lives on a side branch.

## Open questions for Robert

1. Do you have the four font files, or should the stand-ins stay?
2. Phone and tablet: the browser game has full touch controls. Is touch in scope for the Unreal version, or PC and
   gamepad only for now?
