# Note 3: Robert's direction, 2026-11-12 (evening)

What he said, sorted, with where each thing lands in the plans. His words are quoted where they set a rule.

## Decisions

| He said | Lands in |
|---|---|
| Buy the add-ons: "I can totally afford that. So let's go ahead and do it." Named the Action-Adventure Movement System | Note 2. He buys; links given to him |
| Name the five flight styles, "so that we can have a visual difference and a semantic difference" | Below, and Plan 8 ring 4 |
| Three kinds of knock-back: off your feet; knocked back but on your feet; guarded with feet planted "but you feel it" | Plan 8 ring 1 (the three rungs) |
| Diving to the sides, "diving in the different directions" | Plan 8 ring 3 (combat dive along the move direction) and ring 4 (flight dodges: the pack has four directions for each style) |
| Guns held right: pistols, rifles, shotguns, submachine guns, automatic pistols. "I always hated our guns" | Research running; becomes its own plan |
| See the people in the car | Plan 12 (vehicles): occupants visible, seats with hand and foot points already in the vehicle data |
| A training area with the different materials; "I think we should make a punching bag" | Plan 5 and Plan 8: the material wall plus a punching bag and force pads. A good first thing to show what the engine can do |
| "Use Unreal and improve on my vision" | Standing instruction: propose improvements, do not just copy the browser game |
| This is a commercial product | Licences checked on anything reused; 5.9 Preview is for building, not shipping |
| Controller first, still | Plan 14 |

## Questions he asked, answered

| Question | Answer |
|---|---|
| Can I buy the add-ons on another computer? | Yes. Purchases attach to the Epic account, not the PC. They then appear in the library of the Epic launcher on SOVEREIGN |
| What happens to the character models? | They carry over. The blocky hero is already exported for Unreal and Plan 11 builds on it, with its 624 clips and costume system |
| The powers were never assigned | True in his sense: he ruled powers separate from characters (2026-09-26). The 385 roster powers each name a source hero, but who gets what in the real game is still his to assign. Listed as an open call |
| Do we have to figure out speed and the other statistics again? | The rules and numbers come across unchanged. What needs settling is listed in Plan 6 stage S0 (top boost has three figures) and Plan 10 (stat formulas) |
| Should the houses be resized to fit Unreal? | No. One game unit is exactly 19 cm in Unreal, so every house, door and stair keeps its size |
| Keep the blocky style or not? | His call, and it can be made late. Blocks stay the truth for damage and saves either way; what is drawn on top can be blocky or smooth kit pieces (Plan 2, way C) |
| How does destruction work in Unreal, what can we really pull off? | Research running; summary to follow. Plan 3 is the current answer |
| Can we build for virtual reality as we go? | Research running. Short version to follow with the findings |
| Is all of this possible? | Each piece exists somewhere. The combination at this scale is rare. Research running on what is proven and what is new |

## The vision, as he stated it (for every plan to hold to)

- "Imagine Project Zomboid but with superheroes on a galactic scale."
- A different kind of Earth; a smaller version of the world; not every house built by hand.
- Go to the airport and get on a plane. Ships, boats, a water solution, underwater worlds.
- Space is "an intricate part of the system": other planets; light speed or faster outside the atmosphere; a steady speed
  inside the solar system and another outside it; travel shown on the map as something like a comet moving, then you arrive.
- To fly above a certain speed inside the atmosphere, air cannot be one of your needs. Over distance, hunger is the issue.
  (This matches the breath gate in Plan 6: characters who do not need to breathe are immune to it.)
- Your own house, with the builder; order things from your computer in the game.
- Shooting, flying, beams, projectiles, throwing and driving working on a small scale first.
- He is proud of the UI and the grappling system and never got to play them properly.

Added later the same evening:

- **No pop-in.** Far things sit behind a blue haze that thins as they near, as in the real world. Now a measured rule in
  Plan 1.
- **A living world from statistics.** Neighbourhoods and people are generated from statistics; the world has people in
  it. (The browser game has this: the city generator, 8 wealth tiers, the crime index, the seeded civilian generator.)
- **The war game: leaders lead the countries.** They start as NPCs. "You start the server, time goes." So the world runs
  on a server and time passes whether or not you are there.
  **This affects the controller.** A game played on a server needs the hero's movement built for it from the start
  (the player's PC predicts, the server confirms).
  **Answered:** "server client and single player like Project Zomboid." One game, always run by a server; alone, your
  own PC runs it. Plan 8 now builds the controller for this from the first ring, and the roadmap lists what it changes
  in every other plan.
- **Sizing right.** Real dimensions for people and houses; could go photoreal in theory. Real against game sizes are now
  in Plan 2.

Not understood, to ask him: "order from the my field time thing" (which in-game shop or app?), and "we need our mic
thing to be able to send it" (voice notes into the plans?).

## Flight style names (proposal)

The pack has five styles. Measured from the clips: A, B and C fly with no hand leading; D leads with one fist, shoulders
rolled 42 degrees; E leads with both fists. A, B and C differ in ways the measurements do not capture, so their names
are provisional until they are seen in the engine.

| Style | Looks like | Proposed name | What it means in the game |
|---|---|---|---|
| D | One fist forward, body rolled | **Lead Fist** | The striker. Best for swoops and fly-by punches: a hit thrown from this style lands first |
| E | Both fists forward | **Twin Fist** | The ram. Fastest in a straight line, widest turn; made for body contact and breaking through walls |
| A | No hands leading (to confirm) | **Bullet** | Arms pinned. The default cruise: balanced speed and turning |
| B | No hands leading (to confirm) | **Glide** | Relaxed, arms out. Slowest top speed, tightest turn, cheapest on energy: for hovering fights and tight streets |
| C | No hands leading (to confirm) | **Drift** | Upright and loose, as if carried. For characters who float by will or magic, not muscle |

The semantic half is the proposal: each style changes one thing about how flight plays (strike reach, top speed, turn,
energy cost), not just the pose, so picking a style is a choice. The numbers for those differences would go in the rule
file with tests, and each style keeps its own four dodges from the pack. Robert names them; these are a starting point.
