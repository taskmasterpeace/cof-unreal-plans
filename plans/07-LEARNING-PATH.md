# Robert's Unreal learning path

Written 2026-11-12. For learning the engine while the plans are worked. Each step says what the thing is, why this game
cares, and what to look at in OUR project so it is not abstract. No coding needed for any of it.

Epic's free courses live at dev.epicgames.com/community/learning. Search there for the bold words in each step.

| # | Learn this | What it is, in one line | Why we care | See it in our project |
|---|---|---|---|---|
| 1 | **The editor**: viewport, Outliner, Details, Content Browser, Play | Where things are, how to move around, how to press Play | Everything else happens here | Open the slice; press Play; fly |
| 2 | **Actors and components** | Everything in a level is an actor made of parts | The hero, a dummy, a blast and a building are all actors | Click the hero while playing; read its parts |
| 3 | **Character and Character Movement** | Unreal's built-in walking, falling and flying body | Our controller sits on top of it. Its settings are half of "feel" | The hero's movement part: walk speed, braking, flying |
| 4 | **Blueprints vs C++** | Two ways to write game logic: boxes and wires, or text | We use C++ for rules and can use Blueprints for content. You can read Blueprints without coding | Nothing yet; the slice is all C++ |
| 5 | **Animation Blueprints, montages, retargeting** | How a body blends between clips and how clips move between skeletons | It is the next big upgrade to how the hero looks in motion | The 34 combat clips we moved onto Manny |
| 6 | **Nanite and Lumen** | Unlimited-detail models, and light that bounces by itself | Why a city can look good with no baking | Turn Lumen off and on in the slice arena |
| 7 | **Niagara** | The effects system: sparks, smoke, beams | Our beam and blasts are plain shapes today; this is where they get good | The beam in the slice (the "before") |
| 8 | **World Partition and Large Worlds** | How Unreal loads only the part of a huge map near you | The idea behind our cell streamer (Plan 1) | Plan 1, "Scale and coordinates" |
| 9 | **PCG (Procedural Content Generation)** | Rules that place things for you: trees, props, whole blocks | 1,109 cities cannot be placed by hand | Plan 1, the main decision |
| 10 | **Mass (crowds)** | Thousands of simple people and cars at once | The browser game has 80 pedestrians (128 until 2026-11-07). Note 2026-11-13: the trunk's direction is fewer people, each one simulated, so this is learned to measure how many Unreal can afford, not to hit a crowd target, until Robert rules (Plan 12) | Plan 1, the targets table |
| 11 | **Chaos physics and destruction** | Things breaking and falling for real | The look of our destruction (the rules stay ours) | Plan 3, "What changes in Unreal" |
| 12 | **UMG and Common UI** | How menus and the on-screen display are built | All the game's screens get rebuilt here | The slice's on-screen bars (drawn the simple way today) |
| 13 | **Gameplay Tags and the Gameplay Ability System** | Unreal's ready-made system for powers, cooldowns and buffs | A possible wrapper for the 385 powers later; our own rules come first | Plan 10 |
| 14 | **Packaging a build** | Turning the project into a game you can hand someone | How it gets onto another PC | Not done yet |
| 15 | **The Unreal MCP server and Assistant** | The engine's own door for AI to operate the editor | It is how I work in the editor without the mouse | Field guide, section 1 |

## A good first week

- Day 1-2: steps 1-3, with the slice open. Fly it. Change one movement number in the Details panel while playing and feel it.
- Day 3: step 6 and 7. Just watching what they can do is enough.
- Day 4-5: steps 8-10, because they decide how the world plan goes and those are your calls.
- After that: 11 and 12 when the destruction and UI plans start.

## Words you will hear

| Word | Means |
|---|---|
| Level / map | One world file |
| PIE | "Play in editor": pressing Play |
| Cook / package | Make the shippable version of the game |
| Experimental / Beta | Epic's own warning labels. Experimental can change or vanish |
| Preview build | An early engine release. 5.9 is one. Fine for trying things, not for shipping |
| Hitch | One long frame; a visible stutter |
| Draw call | One "draw this" order to the graphics card. The browser game runs out of them; Unreal has far more room |
