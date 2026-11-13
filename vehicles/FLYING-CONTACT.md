# Flying into things: collision, ram, strike, grab

Written 2026-11-13 from Robert's words the same day, for Plan 19 and the controller (Plan 8, ledger J93). Proposal: no
part of it is built in Unreal.

Robert: "There's also momentum... If you fly into something and you're not guarding, that's a collision... I would
think your grit comes into play... If you fly into somebody and you ram, you're guarding during impact. If you fly into
somebody and you punch or strike... Now, you fly and you grab... imagine you're not strong enough... and that stamina,
energy, whatever you want to call it, that they pulling from for that physical thing." And: "Stats and substats are
the heart of the game."

## The one law underneath: the game's own strength contest

The browser game already has it, for tackles and grabs (`_lsw-master/src/data/contact.js` L56-70, L308-339; its
design is the scale doc's B4). It is the right base for flying too, because momentum is already in it:

- **Attack force** = your strength or your momentum, whichever is more (L323). Strength is your lift on the rank ladder
  (L30-37). Momentum force = `mass x closing speed / (0.25 s x 9.81)` (L321): a body absorbs a hit over about a step.
- **Defence force** = your strength, **doubled if braced** (guarding, facing the hit, within 60 degrees), **halved if
  caught** (from behind, unaware, off balance, prone or airborne) (L59-63, L325-327).
- Both become a **score** where one point is one doubling of force (L312). The gap between scores gives the verdict
  (L329, L335-339):
  - **BOUNCE** (the defender is 1.5 doublings or more stronger): the attacker rebounds at 40% of their speed and
    staggers; the defender staggers briefly (L115).
  - **STRUGGLE** (within 1.5 either way): locked together. At speed the pair **ride together**, the one in front
    driven into whatever is ahead first (L111-113).
  - **PINNED** (the attacker is 1.5 doublings or more stronger): the defender is taken, carried or flattened.
  - A body 6 doublings (64 times) stronger than its holder simply is not held (L66).
- The same score sets how long a hold lasts and how fast a held body breaks free (L341-345).

A vehicle enters the contest as a body with mass and no strength: it attacks and defends with its momentum. That is how
"everything the same" (Robert's ruling) works: a person, a car, a jet and a ship meet on one law.

One change is needed for fliers: the browser game counts **airborne as caught**. A hero in **controlled flight** should
count as standing (the game's own brace rule already treats controlled flight as an anchor, `engine/shared-impact.js`
L13-20); only a **tumbling, falling or knocked** flier counts as caught. Proposed; needs Robert's yes.

## The four ways to meet something in flight

| What you do | Its name | How the law reads it | Sub-stats that decide how WELL (never how MUCH) |
|---|---|---|---|
| Fly into it **without guarding** | **Collision** | Not braced. If you did not see it coming (from behind, unaware, tumbling), caught: half force | **Toughness** decides whether the hit knocks you out of flight or down (its job in the sub-stat design: stagger and knockdown thresholds). **Grit** decides whether you keep going hurt: stay in the air, keep fighting, "refuse to fall" (Robert). Grit is the browser design's "fighting hurt" skill |
| Fly into it **with guard up** | **Ram** | Braced: double force. Your momentum or your strength, whichever is more | **Guarding** (brace quality and guard drain). **Power** (how much knock-back you deal and how well you break things: its listed job) |
| Fly in and **punch or kick** on arrival | **Flying strike** | A strike through the damage door, with your arrival speed added (the browser game's momentum melee and dive punch, note 059) | **Striking** (clean hit, timing), **Power** (breaking and knock-back) |
| Fly in and **grab** | **Tackle** (flying grab; Robert: "flying grab would be tackling") | **Against a person or creature: the flight contest of Plan 18 section 6b is the rule** (see "Grabbing a flier" below): you can drive them into the ground and start the ground game. **Against a craft** (car, plane, ship), which has no flight tier: the strength contest decides: BOUNCE, you are torn off; STRUGGLE, you hang on and are dragged, or you drag it; PINNED, you have it: stop it, steer it, throw it. **Front or back only**, read from the approach angle; fatal moves need the back | **Grip** (holding on and breaking holds; its listed signature move is literally "hang from a flier"), **Grappling** (control of the hold), **Lifting** (what you can carry once it is held) |

**Skill can beat rank** (Robert, 2026-11-13: "if you have a skill, it can artificially boost your rank... because
you've practiced it to get better than somebody who was just born with it... it should be able to beat it"). This
overrules the sub-stat design's "skill never beats rank" (`docs/design/SUBSTATS-AND-MAGIC-v0.1.md` L22, L95-102), which
capped a sub-stat at about 15%. Proposed (slice choice, to tune): in the contest, the sub-stat that fits the move adds
up to **2 points of score at 100** (one point is a doubling of force, so a master pushes like someone four times
stronger), scaled evenly from 0. That is enough to turn a lost struggle into a hold against a stronger, untrained body,
not enough to make an ordinary person stop a jet. The sub-stat design document needs the same change; that is for whoever
owns the sub-stats.

## Real speeds change everything

Robert: "I think right now our jets are slow." They are: the browser game's fighter flies 300 u/s (205 km/h) and 485 on
afterburner (332 km/h); a real fighter cruises near 900 km/h and dashes past 2,000. Proposed: real speeds at altitude
(fighter cruise 900, dash 2,000 km/h; airliner cruise 900 km/h at 10.7 km, the game's own schedule figure; helicopters
250-300 km/h), with the flight table re-tuned to match. Momentum grows with speed, so planes become far more dangerous
to meet (the game's contest, guard up; "master" is the skill bonus proposed above):

| Craft | Hero | Guard up | Guard up, master | Unguarded |
|---|---|---|---|---|
| Fighter jet, game speed 205 km/h | rank 50 | STRUGGLE | STRUGGLE | PINNED |
| | rank 60 | STRUGGLE | BOUNCE | STRUGGLE |
| Fighter jet, real cruise 900 km/h | rank 50 | PINNED | PINNED | PINNED |
| | rank 60 | STRUGGLE | STRUGGLE | PINNED |
| | rank 70 | STRUGGLE | BOUNCE | STRUGGLE |
| Fighter jet, real dash 2,000 km/h | rank 60 | PINNED | STRUGGLE | PINNED |
| | rank 70 | STRUGGLE | BOUNCE | STRUGGLE |
| Airliner 250 t, real cruise 900 km/h | rank 70 | PINNED | PINNED | PINNED |
| | rank 80 | BOUNCE | BOUNCE | BOUNCE |

"An ordinary person holding a jet" in the earlier text meant someone with ordinary strength (rank 25 on the game's
ladder): they are flattened in every case, and stay flattened with any skill.

## What the law gives, worked (the game's own contest code, run 2026-11-13, game speeds)

A craft flies into a hero who is hovering. Columns: the hero with guard up, unguarded, and caught out.

| Craft | Hero | Guard up | Unguarded | Caught out |
|---|---|---|---|---|
| Fighter jet 12 t at 300 u/s (205 km/h) | rank 25 (an ordinary person's strength) | PINNED | PINNED | PINNED |
| | rank 40 (lifts 5 t) | PINNED | PINNED | PINNED |
| | rank 50 (lifts 50 t) | STRUGGLE | PINNED | PINNED |
| | rank 60 (lifts 250 t) | STRUGGLE | STRUGGLE | STRUGGLE |
| | rank 70 (lifts 1,000 t) | BOUNCE | BOUNCE | STRUGGLE |
| Helicopter 5 t at 80 u/s | rank 40 | PINNED | PINNED | PINNED |
| | rank 50 | BOUNCE | STRUGGLE | STRUGGLE |
| Car 1.5 t at 110 u/s | rank 40 | STRUGGLE | STRUGGLE | PINNED |
| | rank 50 | BOUNCE | BOUNCE | STRUGGLE |
| Airliner 250 t at 150 u/s | rank 60 | PINNED | PINNED | PINNED |
| | rank 70 | STRUGGLE | PINNED | PINNED |

Read across a row and the guard matters: a rank-50 hero with guard up holds a speeding jet in a struggle; the same hero
caught unaware is flattened. Read down and rank matters most, as the design wants.

## Grabbing a flier: the flight contest (the rule is Plan 18 section 6b)

Robert corrected himself on 2026-11-13: "Why can't you just be flying, run into somebody and grab them and fly into the
ground? Your ability to fly versus their ability to fly has to be taken into consideration. What if you got to use more
power? Do you break loose? Do you take them with you?" The grapple session built it (Plan 18 section 6b,
`FlightContest` in `COFGrappleCore.h`, branch `grapple`). In short, so this document agrees with it:

- **Each side's flight score** = the force it brings in doublings (the grabber's strength or its momentum, the target's
  strength) + its **flight tier** (flyer +1, levitator 0, clumsy -1, cannot fly -1) + the **extra power or ki it
  spends** (up to one doubling, a proposal).
- **What the grabber's lead gives**:

  | Grabber's lead | What happens |
  |---|---|
  | 1.5 or more (overpowering, dominating) | **You drive them into the ground** and land on top: the ground game starts with you mounted |
  | 0.5 to 1.5 (edging) | You hang on; the fight in the air goes on and is contested again |
  | Even | **They break loose** |
  | -1.5 to -0.5 (edged) | **They take you with them**: they fly off carrying you |
  | -1.5 or less (overpowered, dominated) | **They drive you into the ground instead** |

- **Front or back only**, from the approach angle; **fatal moves need the back**, and sneaking works only from behind.
- Once down, Plan 18's ground game applies; a mounted flier can fly off only if its numbers win.

Plan 18 decides grabs between bodies. This document's strength-contest verdicts below apply to rams, collisions and
grabbing **craft** (cars, planes, ships), which have no flight tier of their own.

What each verdict then does in flight (rams, collisions and grabbing a craft):
- **BOUNCE** (the hero holds): the craft breaks where it hit you (its part takes the force through the force gate) and
  deflects; you stagger. A strong braced hero standing in a jet's path is a wall.
- **STRUGGLE**: you and the craft lock together and ride. If you rammed, you are on its nose being pushed; if you
  grabbed, you are hanging on. Every second costs energy (below). Break off, climb along it, punch it, or be thrown when
  your hold fails.
- **PINNED**: you are taken. Knocked back with the craft's speed and hurt by your own change of speed (no cap, Robert's
  ruling), or, if you were the one attacking and won, the craft is yours: stopped, steered or thrown.
- **Punch through**: when your force beats the part's strength by a wide margin the part breaks and you pass through it
  (the game's destruction scale already has the word: `contact.js` L243, "PUNCH THROUGH" at three layers).

## Energy: what holding costs

Robert: "that stamina, energy, whatever you want to call it, that they pulling from for that physical thing."

- **Ki** for powered characters (flight already spends it), **stamina** for operatives and anyone unpowered (Stamina is
  the sub-stat for fatigue and sprinting in the design).
- The cost follows the **load ratio** (what you hold against what you lift), with the game's own three bands for
  lifting: easy up to a quarter of your lift, effort up to 60%, strain up to all of it (`contact.js` L183, L385-391).
  Proposed: a held craft drains nothing extra when easy, a little at effort, fast at strain, and a hold fails when the
  pool is empty. The drain numbers are slice choices for the tuning stage.
- A struggle you are losing drains faster (the break-free rate from the same score, L345).

## Being shot while flying

See "Ideas for a flier being shot" in Plan 19 (research of shipped games and conference talks).

## What Unreal gives for grabs in flight

Checked 2026-11-13 in the 5.9 install (status from each plugin's own file) and on the web. Full notes with links:
the research file `flying-grab-and-shot-reactions.md` in the shared session notes.

| Piece | What it gives a flying grab | Status in 5.9 | With the server deciding |
|---|---|---|---|
| Motion Warping | Bends the reach-and-grab animation so the hand lands on a moving target | Beta | Yes, if server and player set the same target (ours to do) |
| Contextual Animation | Two-character scenes (catching a person in the air) | Experimental | Starts on the server; no prediction, so the player sees lag. Made for people, not vehicles |
| Physics Control | Springs that make the arms strain and the body swing under a wing | Finished | Looks only, on each machine |
| Physical Animation | The older partial ragdoll | Built in | Looks only |
| Chaos constraints | A joint between hero and craft that snaps past a force limit: "torn off" | Built in | Fine on the server; predicting two joined bodies on the player's machine is the hard part |
| Control Rig and full-body IK | Keeps hands on a skid or wing while it shakes | Finished (full-body IK sits in an Experimental folder) | Looks only |
| Mover / ChaosMover | Riding a moving object with rollback | Experimental | Built for it, but unproven on fast fliers |
| Network Prediction | Rollback for movement | Beta | Epic has no answer yet for physics-driven characters the player controls |

**How we build it** (our reading, not Epic's advice): **one decision at the moment of the grab**, on the server, by the
contest above: stop it, steer it, hang on dragged, or torn off. **Dragged** makes the hero a passenger pinned to a point
on the craft (a seat-like grip point), so only one body moves over the network. **Stop or steer** changes the craft's
speed and heading from the hero's input. **Torn off** is the server's timer or force limit (the energy pool running out,
or the contest turning) throwing the hero back into free flight. Hands, sway and the lunge are drawn on each machine
with Motion Warping, Physics Control and IK.

How shipped games did it: Prototype makes grabbing helicopters an unlockable skill (fixed grab points, a quick button
press, hopping from tail to cockpit); Just Cause 3's tethers pull harder with more lines and snap against what will not
move, which is the "not strong enough" case; God of War kept two characters' animations in step for grabs.

## For the board and the sheet

Robert asked that the board be updated with these definitions. The words, ready to copy:

- **Collision**: flying into something without guarding. Toughness decides if it knocks you down; Grit decides if you
  keep going.
- **Ram**: flying into something with your guard up. Braced, you hit twice as hard. Guarding and Power.
- **Flying strike**: punching or kicking as you arrive; your speed adds to the blow. Striking and Power.
- **Tackle** (flying grab): grabbing as you arrive, from the front or the back. Strength, speed and how well each of you
  flies decide it: drive them into the ground, keep fighting in the air, they break loose, they carry you off, or they
  drive you down. Pour in more power to win. Finishing moves only from behind. Grabbing a plane or car: strength and
  speed decide whether you hang on, stop it or get torn off. Grip and Grappling; holding costs ki or stamina.
- **Designations** (the three levels): **Operative** (trained human), **Superhuman** (enhanced), **Living Super
  Weapon**, beside Civilian, Specialist and Creature (`src/data/character-record.js` L25-27).
