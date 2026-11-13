---
title: Consequences of Failure on Unreal
---

# Consequences of Failure on Unreal: the plans

How the game moves from the browser engine to Unreal 5.9. Plans only: nothing in these pages is built yet unless it
says so. Updated 2026-11-13.

## Start here: planes, ships and airports (new)

- **[Plan 19: Planes, ships and airports](plans/19-PLANES-SHIPS-AND-AIRPORTS.html)** — what the browser game already
  has, how flying, ramming, airliners, airports, boats and ships get built, the stages, and your rulings.
- **[Flying into things: collision, ram, strike, grab](vehicles/FLYING-CONTACT.html)** — momentum, guard up or not,
  grabbing a plane, who gives way, which sub-stats decide how well, and what holding costs. Ends with wording for the
  board.
- **[What each damage type leaves behind](vehicles/DAMAGE-LOOKS.html)** — nine damage types in three levels, what each
  does to wood, glass, steel, concrete and snow, how bodies react (flinch, drop, ragdoll), and how it stays cheap.
- **[Ramming: the one law, worked](vehicles/RAMMING.html)** — what happens when a hero hits a jet, a helicopter, an
  airliner or a ship, with numbers.
- **[Water: free plugin or Dynamic Real Water](vehicles/WATER-CHOICE.html)** — what the $99.99 plugin would save and
  what we would lose. Verdict: stay on Epic's free water.
- **[What the browser game has](vehicles/BROWSER-SURVEY.html)** — aircraft, airports, boats, crashes, file by file.
- **[Ready-made on Fab](vehicles/FAB-RESEARCH.html)** — 12 flight and boat products checked.

### Your rulings of 2026-11-13, in one place

1. **One ram law for everything**: weight (mass), speed and strength decide every hit, cars to ships.
2. **A hero can die ramming a plane.** No caps. Guard up against a ramming plane; punch planes.
3. **Shoot from vehicles**: a rider steering a bike fires a pistol at a flier; a passenger fires a submachine gun.
4. **A downed passenger plane is a tragedy**: every death counted, a mass killing if seen, an investigation, funerals,
   and the people who knew the dead find out.
5. **The airliner trip plays in the cabin**; the only skip is fast-forwarding time in single player.
6. **Boats: steal and buy.** Warships live in the war layer as numbers until players are near.
7. **Epic's free water.** Deep oceans and water pressure are ours to build: space (vacuum) and deep water (crush) are
   already separate dangers in the game, so a space flier can dive only as deep as their pressure protection allows.

8. **Stats and sub-stats are the heart of the game**, with powers, abilities, items and gadgets. Every rule reads them.
9. **Flying contact has names**: collision (no guard: Toughness and Grit), ram (guard up), flying strike, and tackle
   (the flying grab: Grip and Grappling; costs ki or stamina).
11. **Skill can beat rank**: practice can win against someone born stronger.
12. **Jets are too slow**: real speeds proposed (900 km/h cruise, 2,000 dash).
13. **Getting shot in flight**: flinch by direction, aim knocked off (Composure steadies it), heavy hits drop you
    (Toughness to drop, Grit to recover), and a chance to ragdoll that varies by damage type.
10. **Every vehicle shows damage by type** (nine types), in the game's style.

### The three levels of character

The game calls them **designations**: **Operative**, **Superhuman** and **Living Super Weapon**, beside Civilian,
Specialist and Creature.

## Every plan

**The order is in [14. The roadmap](plans/14-ROADMAP.html)**, controller first. **What is done is in
[the build ledger](plans/LEDGER.html).**

| Plan | One line |
|---|---|
| [8. The character controller, first](plans/08-CONTROLLER-FIRST.html) | Every way a character moves, the buttons and timings, and the build order |
| [5. Controller obstacle course](plans/05-CONTROLLER-COURSE.html) | Stations, each with a pass or fail number |
| [6. Speed, needs and space](plans/06-SPEED-NEEDS-AND-SPACE.html) | The gates on speed; ground to orbit with no cut |
| [11. Characters and animation](plans/11-CHARACTERS-AND-ANIMATION.html) | Our blocky hero and a real animation setup |
| [10. Powers and combat](plans/10-POWERS-AND-COMBAT.html) | 385 powers on about 55 behaviour types |
| [15. Guns, held right](plans/15-GUNS.html) | Each weapon class gets its own hold; dive-and-shoot; people in cars |
| [9. Screens and the on-screen display](plans/09-SCREENS-AND-HUD.html) | One palette, 30+ screens, in order |
| [2. Buildings and the house formula](plans/02-BUILDINGS-AND-HOUSES.html) | Block buildings; walk the hero through 1,920 houses |
| [3. Destruction](plans/03-DESTRUCTION.html) | Same damage rules; falling floors, debris, fire and light |
| [1. The world](plans/01-WORLD.html) | 1,109 cities from a seed; our own cell streamer |
| [12. People, police and vehicles](plans/12-PEOPLE-POLICE-VEHICLES.html) | People, six police levels, vehicles |
| [13. Saves and sound](plans/13-SAVES-AND-SOUND.html) | Saves without the browser's limits; 3D sound |
| [16. The server and the war game](plans/16-SERVER-AND-WAR-GAME.html) | Solo, friends and community servers; the war layer; 198 countries |
| [19. Planes, ships and airports](plans/19-PLANES-SHIPS-AND-AIRPORTS.html) | Flying, ramming, airliners, airports, boats, ships, consequences |
| [4. Tools and data](plans/04-TOOLS-AND-DATA.html) | One export gets all game data out of the browser code |
| [7. Learning path](plans/07-LEARNING-PATH.html) | 15 engine topics in the order the game uses them |
| [17. How we build it](plans/17-HOW-WE-BUILD.html) | The working loop and short prompts |
| [14. The roadmap](plans/14-ROADMAP.html) | Everything in one order |

Research notes: [engine facts](plans/R1-ENGINE-FACTS.html) ·
[existing controllers](plans/R2-EXISTING-CONTROLLERS.html) ·
[your direction, 2026-11-12](plans/R3-ROBERT-DIRECTION-2026-11-12.html) ·
[what is possible](plans/R4-WHAT-IS-POSSIBLE.html) · [trunk reconcile](plans/R5-TRUNK-RECONCILE.html) ·
[corrections from real runs](plans/R6-EXECUTED-CORRECTIONS.html)

## The method

Every rule lives in a file with no engine in it and is tested against results recorded from the browser game. Only
then does Unreal draw it. Nothing here is built until its ledger row says DONE with proof.
