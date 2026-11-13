# Plan 4: Tools and data

Status: PLAN ONLY (2026-11-12). Nothing here is built.

## The finding that shapes this plan

Most game data is inside the browser game's code files, not in data files. Unreal cannot read it as it is. But those files
load in plain Node, so one export script can write all of it out. The browser tools (96 pages) mostly edit that same data,
so they keep working as the editors while Unreal becomes a second reader.

**Rule: one truth, two readers.** Data is edited in one place (the existing tools) and exported to a neutral folder that
both the browser game and Unreal read. Nobody retypes a number in Unreal.

## The data to export (stage T0, no Unreal needed)

| Data | Source file in `D:\dev3\lsw\src\data` | Records |
|---|---|---|
| Cities | `cities.js` + `citycoords.js` | 1,109 (1,047 surveyed, 62 typed by hand and marked as estimates) |
| Countries | `countries.js` (the last 30 rows come from `countries-30.js`) | 198 (25 fields) |
| Borders, coast | `borders.js`, `earth.js` | 178 countries, 18,576 points |
| Powers | `library/powers-roster.js` | 385 |
| Roster characters | `characters.js` | 66 |
| Weapons and gear | `armory.js` | 16 firearms, 7 blades, 23 gear, 9 loadouts |
| Items | `item-catalog.js` | 32 categories |
| Creatures | `creature-catalog.js` | 83 |
| Vehicles | `road-vehicles.js` | 11 (26 variants) |
| Damage types | `damage-types.js` | 9 |
| Threat scale | `library/lefevre.js` | 11 rungs |
| Materials | `voxel-spec.js` | 58 |
| Building types | `city-recipe-catalog.js` | 391 catalogue entries on the trunk (the game's own document still says 119; 88 are new generated-house entries) |
| Weather | `weather.js` | 14 states |
| Already JSON | `sound-picks.json`, `move-kits.json`, `gait-table.json`, `hud-components.json`, `ui-themes.json`, clip banks | as they are |

Output: `D:\dev3\cof-unreal\Data\*.json`, one file per table, with a manifest listing each file's record count and the
commit of the game repo it came from. Unreal imports them as Data Tables with a Python script (run once per export).

## Assets

| Asset | Count | Plan |
|---|---|---|
| Blocky hero | 1 model, 53 bones, 152 parts, 36 costume slots | Already exported to FBX in `SourceArt/Hero`. Import, build the part on/off system from the manifest's slot rules |
| Hero animation | about 624 clips in 16 banks | Convert with the same Blender exporter; they are already on the hero's own skeleton, so no retarget |
| Purchased packs | about 4,700 Unreal files | Already in the project; 34 combat clips already moved onto Manny |
| Props | 856 models, with damaged versions | Batch convert to FBX and import as static meshes |
| Sounds | 1,921 MP3 + 1,384 WAV | Import WAV as they are; MP3 converted to WAV at import. The 23 picked event sounds first |
| Power icons | 678 | Import as textures when the UI starts |
| Raw mocap | 11.8 GB on the NAS | Leave on the NAS; pull clips by need |

## Tools, in the order they pay off

| # | Tool | What it is | Needs Unreal? |
|---|---|---|---|
| T0 | **Export script** | Node script: every table above to JSON, with counts checked | No |
| T1 | **Golden exporter** | Same idea for behaviour, so the C++ ports can be tested against the browser game: 6 cities, 1,920 houses, 12 buildings, 120 damage cases, 13 collapses, one 10-minute scripted flight, movement and knock-back cases, about 98 power cases, a scripted crime spree for police heat, and sample save files (feeds Plans 1, 2, 3, 6, 8, 10, 12, 13) | No |
| T2 | **Standalone test runner** | One command builds and runs every engine-free test set (controller 113 today; world, house, blocks to come) | No |
| T3 | **Import script** | Python in the editor: JSON to Data Tables, FBX batch import | Yes |
| T4 | **COF toolset for the editor's MCP server** | Our own commands an AI can call in a running editor: spawn enemy, set power, build house from seed, damage block, run course, read telemetry | Yes |
| T5 | **Course runner** | The autopilot from the slice, grown into a station-by-station runner with a PASS/FAIL sheet (Plan 5) | Yes |
| T6 | **Proof shots** | Fixed camera spots per station and per city, same picture every build, to catch looks breaking | Yes |
| T7 | **Tuning panel** | Sliders for the controller numbers in the running game, saved back to the one tuning file | Yes |

## Which browser tools to keep, rebuild or drop

| Browser tool | Verdict | Why |
|---|---|---|
| Power Forge, Item Forge | Keep; add "save to file" | They save in the browser only today. Unreal needs a file |
| Sound Board | Keep | Already writes a JSON file |
| Floor Plan Lab, House generator | Keep as the reference; add golden export | The Unreal house must match it |
| Atlas, Sector Manager | Keep as the reference for the world port | |
| Character editor, Fitting room | Keep for recipes; the look gets rebuilt in Unreal once the hero is imported | |
| Animation Studio | Keep until the hero's clips are in Unreal, then Unreal's own animation editor takes over | |
| Grip Lab | Replace with sockets in Unreal | Nothing in the game reads its file today |
| Car lab | Viewer only; replace when vehicles are ported | |
| Threat Room, Training Hall | Rebuild as the obstacle course (Plan 5) | |

## Which version of the game the exports come from

The game folder on SOVEREIGN is sitting on a side branch with unsaved changes. Exports and goldens must come from a named
commit on the main branch, written into the manifest, so a number can always be traced back. The export script checks out
that commit into a scratch copy; it never touches the working folder.

## A loose end found by the survey

The manifest of tools that save data exists only in an unmerged side branch of the game repo. It lists 18 tools with where
each saves. It should be merged before the export script is written, because the script needs that list.

## Stages

T0, T1 and T2 can start today with Unreal closed, and they unblock the first stages of Plans 1, 2 and 3.
