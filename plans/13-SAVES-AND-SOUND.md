# Plan 13: Saves and sound

Status: PLAN ONLY (2026-11-12). Nothing here is built.

## Saves

**Today:**

| Save | Holds | Where |
|---|---|---|
| Player | hero, cell, position, health, needs, custody; on the trunk also death records and bodies, whether flying, the time of day, and the nearest city cell to reopen in. Version 1. Autosave every 20 s | `world-cell/player-save.js` |
| World | one file per 256 m cell, capped at 8 MB: the block damage journal (compressed), collapses, cars, items, households, economy; on the trunk also tree damage, hung doors and objects placed in a home | `world-cell/checkpoint.js` |
| Registry | recipes, never geometry | `src/data/world-registry.js` |
| Heroes made in the creator | a list in browser storage | `src/data/creator.js` L404 |

Browser storage holds about 5 million characters, then falls back to a second browser store or the play server. That
limit is the main weakness.

**In Unreal:** the same shape, without the limit. One player file, one file per changed cell, written off the main
thread through the engine's raw save slots. The engine's only world-state framework is Experimental and tied to
hand-made levels, so we keep our own format (research note 1).

**Where saves live** (Robert, 2026-11-12): the game is server and client, with single player as your own PC running the
server. The world save and every hero live on the server where they were made. A hero does not travel between servers;
joining another server means making a hero there. In single player the "server" files are simply on your own PC. On a
server someone else hosts, this closes the usual cheating route of editing a local character file. The host can always
edit their own server's files, as in Zomboid; official season servers are the answer to that (Plan 16).

Rules to keep: a save stores recipes and changes, never geometry; cell ids stay the same as the browser game; every file
carries a version number.

| # | Stage | Needs Unreal? | Done when |
|---|---|---|---|
| V0 | Write the save format down as a spec, from the browser game's code | No | Every field in the table above has a type and a version |
| V1 | Reader and writer in the engine-free file, with tests | No | A browser-game cell save loads and writes back byte-for-byte equal after decompression |
| V2 | Player save in Unreal | Yes | Quit and restart: same place, health and needs |
| V3 | Cell saves with block damage | Yes | Break a wall, restart: same hole (Plan 3, stage D7) |
| V4 | Import a browser-game save | Yes | A hero and a damaged city made in the browser open in Unreal |

Open question for Robert: should browser-game saves carry over into the Unreal game (V4), or is a fresh start fine?

## Sound

**Today** (`src/core/audio.js`): five channels (music, effects, voice, ambience, interface) into a compressor, with ducking.
Position is left-right pan plus distance, not true 3D. Music is generated, plus car radio. Civilian voices are
synthesised; regional voices are recorded; the hero's voice is off by default. Material hits and footsteps are built, and
so is ambience by district, culture and time of day. Layered gunfire (first shot, body, tail, last shot) is built for
4 gun families but off by default.

Assets: about 3,000 sound files (the critic counted 3,082 in the game's public folder; the survey's 3,305 included
other folders); 21 to 23 game events have a picked sound, to be counted in the export; 9 impact materials; 11 district
ambiences.

**In Unreal:**

- True 3D position, sound blocked by walls, and echo by room size all come with the engine. The block buildings give the
  room sizes for free.
- The five channels become five sound classes with the same starting levels (music 0.34, effects 1.0, voice 0.92,
  ambience 0.52, interface 0.7) and the same ducking.
- Gunfire layering follows the studio's gun-audio rules and is on from the start.
- Synthesised civilian voices: the engine's own sound synthesis tool can do this; unproven. Fallback is recorded lines.

| # | Stage | Needs Unreal? | Done when |
|---|---|---|---|
| N0 | Export the sound picks, material table and ambience table; convert the picked event sounds (Plan 4) | No | Picked events counted; 9 materials; 11 ambiences |
| N1 | Controller sounds: footsteps by material, take-off, boost gears, wind by speed, punches, blasts, the beam, landing, crash | Yes | Each of the controller's moves makes its sound in a recorded pass; none missing in the log |
| N2 | Five channels with ducking | Yes | Voice lowers music by the game's amount, measured |
| N3 | Destruction sounds by material | Yes | Breaking each of the 9 impact materials plays that material's own sound file, shown in the log |
| N4 | City ambience by district and time | Yes | Walk from downtown to suburb at noon and at night: the log shows 4 different beds |
| N5 | Guns, voices, music | Yes | Each of the 4 gun families plays first shot, body, tail and last shot in order in the log; a voice line lowers music by the game's ducking amount; music plays and crossfades once |
