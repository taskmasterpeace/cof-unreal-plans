# Research note 4: what is really possible

Written 2026-11-12. Robert: "Is it possible? ... it feels like we're trying to do something that's never been done
before." And: "Can we build it for virtual reality as we're building it?" And: "I don't know how this destruction system
works in Unreal... what can we realistically pull off?"

Read from the 5.9 install and the web; nothing was launched, bought or downloaded. "From memory" means not re-checked today.

## Is it possible?

Each piece exists somewhere. Nobody has shipped all of them together, in any engine.

| Hard thing | Who has done it | In Unreal? |
|---|---|---|
| Flying from the ground into space with no cut | Star Citizen, No Man's Sky, Outer Wilds | Not in a major shipped Unreal game that the search found |
| A generated city where every building can be entered | Shadows of Doubt | Not found in Unreal |
| Breaking any block of any building | Teardown, Minecraft | Not as blocks. One Unreal shooter does large pre-broken destruction |
| Thousands of people and cars | Epic's Matrix city demo | Yes, as a demo. The crowd system is still marked Experimental |
| Fast superhero movement over an open city | Spider-Man, Saints Row IV | Partly: one Unreal 4 superhero game has fast city traversal, not supersonic and not to space |

So he is right that the combination is new. That does not make it impossible. It makes the order matter: it is realistic
only as a staged build where each stage is a game you can play, which is what the roadmap does. The controller, a
training area, one house, one city, then outward.

Boats, airliners, underwater worlds and VR are each big features by themselves. They belong after the core plays well.

## Destruction: what Unreal offers, in plain terms

| Way | Looks like | Hole anywhere? | Saves? | Works for a whole city? |
|---|---|---|---|---|
| Unreal's built-in destruction (pre-broken models) | The most realistic chunks. Each object is cracked in advance in the editor | No. It only breaks along the cracks made in advance | Not built in | No. Fine for a few special buildings |
| Cutting models apart during play | Same look, cut on the spot | In theory | No | No. The tools exist in the engine but nobody documents using them in a shipped game |
| Our blocks, re-drawn when hit | Blocky, like the game today | Yes | Yes, as a small list of changes | Yes |
| Blocks for structure, Unreal's physics for loose debris | Blocks that break anywhere, with real chunks flying, falling and settling | Yes | Blocks yes; loose debris is not saved | Yes |

**Recommendation: the last one.** Blocks decide what is standing. Unreal's physics makes the pieces that fly off look and
feel real, and they are cleaned up after a while. That is Plan 3.

**On "broken blocks never fall":** in Unreal, both rules are affordable. Pieces that lose their support can fall as real
chunks, which the browser could not do. So the choice is about how the game should feel, not about cost. A fair way to
decide it: build one test wall and one test house in the training area that can switch between "blocks never fall" and
"what loses support falls", and he picks by playing.

A smooth-terrain voxel tool exists for Unreal ($349, still in preview, lists 5.6 and 5.7 only). It dropped cube-style
blocks, so it does not replace our block buildings. It could be useful later for terrain and craters.

## Virtual reality

**Keep the door open; do not build it alongside.**

What carries over to VR unchanged: the rules, the world, the powers, enemy brains, saves, sound.

What has to be redone for VR:

| Thing | Why |
|---|---|
| Every menu and on-screen display | Flat screen overlays do not work in a headset; each needs an in-world version |
| The camera | A chase camera behind a body flying fast is the worst case for motion sickness. Shipped flying VR games use a cockpit frame, a tunnel effect at speed, and snap turning |
| Controls | Hand controllers instead of keyboard or pad |
| The speed budget | A headset needs a locked 90 pictures a second in both eyes: 11 ms a frame against 16.7 ms flat, with about twice the pixels. On SOVEREIGN's card that means lower lighting quality, fewer people, shorter view distance |

Unreal's VR support itself is production quality and ships with a starter template. Its high-end lighting and detail
systems do run in VR on PC, but Epic's own template turns them off, and one 5.8 version crashed at VR start until a patch.
Not tested on 5.9.

Three cheap habits now keep VR possible later, and they are good practice anyway:

1. No game rules inside the camera or the display. (Already how the slice is built.)
2. Build display panels so they can be drawn on a surface in the world as well as on the screen.
3. Keep one quality dial that scales lighting, crowds and view distance.

There is precedent: flat Unreal games have shipped free VR versions later, and a fan tool puts flat Unreal games into a
headset with no work from the developer.

## Water, boats, aircraft

| Want | What the engine has | What we would add |
|---|---|---|
| Ocean, lakes, rivers | A water system (Experimental): waves, shorelines, an underwater screen effect, floating objects | It is a flat ocean inside a bounded area. A round planet's ocean is ours to build |
| Underwater worlds | Only the screen effect and basic swimming | Underwater light, seabed, creatures: a feature of its own |
| Boats | Nothing for boats; the vehicle system is wheels only | A floating body with our own engine and rudder forces |
| Aircraft | No aircraft or template. There are wing and thruster parts in the vehicle system (Experimental) | Our own flight model |
| Airport, board a plane | Standing on a moving deck and sitting in a seat are standard | The usual way for airliner trips: the cabin is its own small level and the outside is faked. Far cheaper than simulating a walkable cabin at 900 km/h |

Three ocean products are sold for Unreal, from about $70 to $350; none is confirmed for 5.9 and the prices were not
confirmed on their store pages.

## What this changes in the plans

- Plan 3 gets a switchable collapse test in the training area, so the two rulings can be compared by playing.
- Plan 9 gets one rule: every display panel can be drawn in the world as well as on screen.
- Boats, aircraft, the airport and underwater worlds move from "not planned" to "planned after the core, each as its own
  plan", with the cabin-as-its-own-level approach for airliners.
- VR is recorded as "later, door kept open".
