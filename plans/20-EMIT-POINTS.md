# Plan 20: Where attacks come out of the body (emit points)

Written 2026-11-13 (jobs J190-J196). Machine King, 2026-11-13: *"we got to figure out all of the different ways to emit
attacks... like it's through the eyes, just to start off."*

Source: the export copy `_lsw-master` of the browser game's trunk (commit in `Data/manifest.json`). Paths below are
relative to `_lsw-master/src`. Note: the powers list is `data/library/powers-roster.js` (not `data/powers-roster.js`,
which does not exist on the trunk). It holds 385 powers with an attack: dash 55, buff 47, projectile 45, melee 39,
cone 38, beam 24, charge 25, construct 17, rifle 15, rush 15, volley 13, teleport 10, meteor 9, and smaller types
(counted by loading the file in Node).

## 1. What the browser game does today

**Only three body points are chosen by data: hands, chest and eyes.** Everything else is fixed by the power's type or
comes from a held weapon.

The one function: `Fighter.muzzle(out, fwd, h)`, `engine/entity.js:592-603`. It picks the point from the height
argument, not by a name:
- `h === 8.3`: **eyes**, the midpoint of the `eyeL` and `eyeR` parts, plus 0.12 u along the aim (`:594-597`).
- `h <= 5.4`: **chest**, the `torso` part plus `fwd` along the aim (`:598-600`).
- any other `h`: **hand**, `armR.children[2]` plus `max(0, fwd-3.4)` along the aim.
- No rig: `pos + aim*fwd` at height `h` (`:602`).

The data switches:

| Switch | Where | Picks | Roster powers using it |
|---|---|---|---|
| `faceOrigin: true` | `data/attack-tuning.js:158`, read `:61`, `:66` | eyes | 2: `heat-ray` (`data/library/powers-roster.js:7`), `eye-beam` (`:94`) |
| `chest: true` | `engine/power-emission.js:37` | chest | 1: `reactor-burst` (`:116`) |
| `castHand` | `engine/hand-emission.js:6` | left or right hand | 0 set; default hand used |
| `castStyle` (`auto`, `palm`, `two-hand`, `optic-focus`, `chest-brace`) | `data/attack-tuning.js:42-46` | the pose; `two-hand` = both palms | 0 set |
| `emissionOrigin` (`auto`, `right`, `left`, `paired`, `chest`, `eyes`) | `data/attack-tuning.js:47-63` | the tuning tool's one switch, compiled into the three above | tool only |
| `handPattern` (`paired`, `left`, `right`, `alternate`) | `engine/hand-emission.js:11-18` | which hand each volley shot leaves | default `alternate`; 4 `oneHand` |
| `weapon` (`rifle`, `shotgun`, `pistol`) | `engine/weapon-emission.js:12-50` | the gun's `weapon-muzzle` node, else the hand; two guns alternate (`:10-11`) | 15 rifle-type |
| `groundslam: true` (nova) | `engine/abilities.js:1366` | the ground under the hero | 1: `world-breaker` (`:170`) |
| `naniteForm: 'cannon'` | `engine/power-emission.js:31-33`, `engine/nanite-forearms.js:271-277` | a forearm cannon | not in the roster |

Outside the roster the hero creator has `opticblast` (eyes, `data/creator.js:144`) and `unibeam` (chest, `:149`).
`data/visual.js:62-84` lists nine "source" words (body, hand, face, chest, weapon, ground, sky, aura, world) but they
only colour the look; nothing is placed by them.

**Fixed by type** (`engine/abilities.js`):

| Type | Comes out of | Line |
|---|---|---|
| projectile | right-side hand, `muzzle(3.6, 5.8)` | 513, 530 |
| volley | the real hand or hands, alternating | 547; `engine/projectiles.js:661` |
| beam | hand, or eyes / chest by the switches; re-sampled every frame | 588; `engine/projectiles.js:1784-1793` |
| cone (breath, scream, stomp) | hand, `muzzle(3.0, 6.2)`, for the look only; the damage arc is centred on the body | 614-620 |
| charge | hand, or chest; launch re-resolved, eyes allowed | 729, 761 |
| facebomb | hand, `muzzle(3.6, 6.4)` | 1306 |
| rifle / bow | the gun's muzzle / the bow's `bow-launch` node | 1218, 1018; `engine/bow-pose.js:7-11` |
| nova | body centre (`pos + 5`), or the ground with `groundslam` | 1363-1384 |
| ground spikes | a line along the aim from the feet | 649 |
| mine | at the aim point, capped by range | 1500-1503 |
| meteor | the sky, 90 u over the aim point | 1562 |
| growing orb | over the head, `pos.y + 16 + radius` | `engine/projectiles.js:2543` |
| summon | at the hero | 858-859 |
| grenade (soldier) | a prop in the left hand; released 1.6 u ahead of the chest or over cover | `engine/throwable-action.js:20-30`, `engine/grenades.js:169-180` |
| thrown prop | the carried thing, else `muzzle(5, 6.4)` | `engine/thrown-prop-contact.js:16` |
| melee | the fist, or the weapon's sweep | `engine/melee-contact-limb.js:37-40`, `engine/ability-melee-contact.js:22-28` |

**Aim:** a beam points from the emit point to the aim point, turning at its `steer` rate (`engine/projectiles.js:1873-1900`),
not along the eye line; the hands must be braced and facing the target before it launches (`:1797-1810`). Eyes and chest
claim no hands, so the hero can still use them (`engine/cast-channels.js:6-24`). Eyes put one hand to the temple
(`optic-focus`); chest braces both fists (`engine/combat-pose.js:291-315`).

**Trap: left and right are swapped in the browser rig.** `armR` is the body's anatomical left
(`engine/hand-emission.js:3-5`), so the game's default hand is the left one and `castHand: 'right'` means `armL`.

**Powers whose name promises a place the code does not use** (the "improve on my vision" list):
- Breath and shouts come out of the hand: `arctic-breath`, `frost-breath`, `tempest-breath`, `siren-scream`,
  `apex-roar`, `shield-shout`. There is no mouth point anywhere.
- `unibeam-ironclad` (`:159`) is a beam with no chest switch, so it fires from the hand.
- `shockwave-stomp` (`:296`) is a cone from the hand, not the ground.
- `tail-sweep` and `sting-kick` hit through the hand node; there is no tail or foot point for powers.
- Wing powers (`wing-ding-fan`) leave the hands; there is no back point.

## 2. Our hero today

Every blast, ball and beam leaves one place: the right hand bone (`ACOFHero::HandLocation`, `hand_r`); without art,
45 cm ahead and 30 cm up. No other point exists.

## 3. The proposal: named emit points on the hero

Twelve named points. Body points are sockets on the skeleton (Manny's bone names, so any body retargeted to it gets
them free); the last four are places in the world worked out by the rule.

| Point | Socket on | Used by (browser) | Used by (new, our call) |
|---|---|---|---|
| `Eyes` | `head`, midpoint between the eyes, a little forward | `faceOrigin` beams and charges (heat ray, eye beam, optic blast) | any power tagged eye or optic |
| `Mouth` | `head`, at the mouth | (none: breath came from the hand) | breath, scream, roar, shout cones |
| `Chest` | `spine_05`, front of the sternum | `chest` (reactor burst, unibeam) | `unibeam-ironclad` |
| `HandR`, `HandL` | `hand_r`, `hand_l`, the palm | default and `castHand` | |
| `Palms` | midpoint of both palms (worked out, not a socket) | `two-hand`, paired volleys | |
| `Muzzle` | the held weapon's own `Muzzle` socket, else the hand that holds it | guns, two guns alternate | |
| `ForearmR`, `ForearmL` | `lowerarm_r` / `lowerarm_l`, top of the wrist | nanite cannon | wrist launchers, gadget darts |
| `Back` | `spine_03`, between the shoulder blades | (none) | wing fans, missile packs (`micro-missiles`) |
| `Feet` (ground under the body) | `root` projected to the floor | `groundslam`, ground spikes | `shockwave-stomp`, other stomps |
| `Body` | pelvis / capsule centre | nova, aura, summon | |
| `Overhead` | above the head | growing orb | |
| `Sky` and `AtTarget` | world points: over the aim point / at it | meteor; mine | |

**How a power picks its point** (one rule, in order; the first that answers wins):
1. The power's own `Emit` field, if set (new; one word from the table).
2. The browser switches, mapped exactly: `faceOrigin` -> Eyes, `chest` -> Chest, `emissionOrigin` -> its word,
   `castStyle: two-hand` -> Palms, `castHand` -> that hand (mapped to the **anatomical** side; see the trap above),
   `weapon` -> Muzzle, `naniteForm: cannon` -> Forearm, `groundslam` -> Feet.
3. The type's default from the table in section 1.
4. Our fixes for the name-promise list (Mouth for breath and shouts, Chest for the ironclad unibeam, Feet for the stomp,
   Back for wing fans), behind one switch `EmitAsBrowser` so the exact browser behaviour can be checked against cases.
   Default: the fixes are on (Machine King: "use Unreal and improve on my vision").

**Which hand:** default hand is the hero's right (slice choice: guns are held right, Plan 15; the browser default is the
rig's `armR`, which is the anatomical left). Volleys alternate by default, `paired` fires both, `oneHand` stays on one.

**Aim:** every point fires from the point toward the aim point, as the browser does. A point that cannot see the aim
point (eyes when the head is turned away more than the neck allows) turns the head first; the beam does not bend out of
the back of the skull. Eyes, Mouth, Chest and Back leave both hands free; the hands get the browser's poses
(hand to temple for eyes, braced fists for chest).

**What carries over:** the rule is engine-free (`COFEmitCore.h`: power data in, point name and offset out); the
socket offsets live in one table so a new body only fills that table.

## 4. Machine King's widening (2026-11-13, later) and the calls made on it

His words: *"you can emit projectiles from the fist, like from one fist while they're flying, in one direction, going in
another direction, throwing something, shooting something... living super weapons won't be able to shoot while they're
flying in the sky... if you can fly, you can't be superhuman or an operative. You're a living super weapon... that's our
secret sauce... the eyes, the finger... palm, the palm being up, both palms, shooting two guns... it emits from all
around your body... a beam coming out of your actual body in the shape of your body going forward, or wherever you aim...
eye, finger, both palms, both fists, one fist... characters that have limbs disabled... carrying things with one arm,
carrying things with both arms. They end up dropping it if they attack, or do they have to drop it before they attack?
You decide."*

So an attack is three separate answers, not one: **where** it leaves the body (the point), **what shape** it takes, and
**what the body is doing** while it fires (flying one way, carrying, missing an arm). Each is its own small rule in the
engine-free file, so they combine freely: an eye beam while carrying a car one-handed while flying away from the target
is three answers, not a special case.

### 4a. Where (more points)

Added to the twelve in section 3:
- **`FingerR`, `FingerL`**: the index fingertip (`index_03_r` / `index_03_l`). The pointing shot: a finger bolt, a
  thin beam, a finger-gun. Owned clips: the Pose Animations pack's 14 pointing poses (`A_Point01` to `A_Point14`, most
  with a mirrored twin).
- **`PalmUp`**: above an upturned palm. Not a shot forward but a thing raised: an orb that grows on the hand, a flame
  held up, a call to the sky (meteor and lightning powers land at `Sky` but start with the `PalmUp` pose and effect).
- **`FistR`, `FistL`**: the knuckles, separate from the palm, because a fist shot and a palm shot are different moves
  (owned: the Kung Fu pack's double fist punch and flick punch, and its double palm push).
- **Silhouette** and **Surface** use the whole body; they are shapes, below.

### 4b. What shape

| Shape | What it is | Example |
|---|---|---|
| Bolt | one thing leaves the point and flies | ki blast from one fist, finger bolt |
| Beam | a held ray from the point to the aim, steered | eye beam, palm beam |
| Twin | two of the same from a pair, side by side | both palms, both fists, both eyes, two pistols |
| Alternate | a pair taking turns | volleys, two guns in a run |
| Cone or spray | a widening wedge | breath, scream, a fan of finger rays |
| **Silhouette beam** | a beam shaped like the hero's own outline, pushed forward along the aim from the whole body | "a beam coming out of your actual body in the shape of your body" |
| **Surface burst** | out of the whole body at once, every way | "emits from all around your body": a nova that starts at the skin, not a ball from the belly |
| Ground line | runs along the floor from the feet | stomps, spikes |
| Drop | lands on a place | meteor (Sky), mine (AtTarget) |
| Throw | a held thing leaves the hand with the hero's speed added | thrown car, grenade |

The silhouette beam is a wide beam whose cross-section is the body's: its hit test is a box swept along the aim, the
hero's height by shoulder width (slice choice: the capsule's height by twice its radius), and its look is the hero's
outline stretched forward. It is the one beam that turns the whole body to face the aim before it fires.

### 4c. What the body is doing

**Flying one way, firing another.** The legs keep flying the way the body is moving (J190). The upper body turns to the
aim up to a limit (slice choice: 100 degrees left or right, 70 up or down), using the owned aim poses (the Pose
Animations pack has 14 aim sets of 9 directions each) and the flight pack's leans. Past the limit the whole body rolls
to face the aim while still travelling the old way: firing behind while flying forward is a roll onto the back.
Hands-free points (eyes, mouth, chest, back, surface) turn with the head or chest only, so an eye beam can track sideways
without the arms moving.

**Guns are not for a living super weapon in the air (Machine King's rule).** A hero that can fly does not fire guns
while flying; guns are for operatives and superhumans, and for a flier only with its feet down. The gun muzzle point is
not offered in the air to a flier; gun-like powers (finger-gun, two-palm shots) are. Two pistols use the Twin and
Alternate shapes like any pair.

**The secret rule: flight makes a living super weapon.** Any hero that can fly (any flight level above "grounded",
`data/creator.js:49-54`) is filed in the Superweapon class with a threat of at least Very High on the LeFevre scale
(`data/creator.js:23`, `:40-46`), whatever its points. In the browser game flight is a trait any budget can buy
(8 to 22 points), so this is a change. The player is never told why; they only see the threat.

**Carrying. Machine King left this call to us: the hero drops it as the attack starts. There is no separate drop
step.** In order:
1. A hands-free point (eyes, mouth, chest, back, surface, silhouette, feet) fires and the hero **keeps holding**. Heat
   vision while holding a car over your head is the picture.
2. Carrying one-handed, a one-hand shot fires from the **free** hand (a finger or fist shot with the left while the
   right holds the crate).
3. If the attack needs the carrying hand (both palms, both fists, a gun during a two-handed carry, any hand shot while
   carrying with both arms), the hero **lets go on the attack's first frame** and the thing falls with the hero's speed.
   From the sky it falls from the sky: a consequence, which is the game's name. No button to drop first: fewer presses,
   and it reads as the character choosing to fight.
4. **An attack never drops a carried person.** A hand attack that would is refused, with the reason on screen ("Hands
   full: you're holding someone"), the way the browser refuses two-hand powers while hanging one-handed from a ledge
   (`engine/abilities.js:1615-1624`). The hands-free points still work.
5. Throwing is its own attack: the carried thing becomes the shot.

**Lost or busy arms.** A disabled arm, a hand on a ledge or grapple, and a hand holding something are one idea: a
**busy hand**. The rule is told which hands are free and moves the shot. One-hand shots go to the other hand (the
browser already does this for power punches when an arm is disabled, `engine/ability-melee-pose.js:20`, `:37`). Pairs
(both palms, both fists, two guns) become one-hand versions if the power allows, else they are refused with the reason.
With both arms gone only the hands-free points work. In the browser game disabled limbs exist only for zombies
(`engine/zombie-locational-damage.js:17`); our wounds already track each arm (the damage rule's six zones), so a badly
wounded arm can count as busy.

### 4d. What we own to animate it (file names read locally; nothing uploaded)

Finger pointing (Pose Animations: 14 point poses), aiming in nine directions (Pose Animations: 14 aim sets), double
fist, double palm and flick punch (Kung Fu pack), a ranged cast and a special (Evil Magician pack), flight leans and
poses in five styles (Superhero Flight pack). Not owned yet: carry holds (to find when J91's holds are built). Eye,
mouth, chest and surface shots need no arm clip: a head or chest turn and the effect.

## 5. Jobs

J190 is the beam-pose bug. J191 is its arm layer. J192 to J196 build the points; J197 to J200 build section 4. See the
ledger.
