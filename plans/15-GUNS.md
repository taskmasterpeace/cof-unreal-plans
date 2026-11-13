# Plan 15: Guns, held right

Status: PLAN ONLY (2026-11-12). Robert: "I always hated our guns... They don't hold our guns right in that web version at
all. We need our pistols, rifles, shotguns, submachine guns, automatic pistols being held right." Also: diving to the
sides while shooting, and seeing the people in the car.

## Why the guns look wrong in the browser game

The game has 16 firearms in six classes, and recorded holds for only two of them.

| Class | Guns | Its own recorded hold? |
|---|---|---|
| Pistol | 3 (one is the machine pistol) | Yes: 9 takes (idle, aim, five aim pitches, shot, reload) |
| Rifle | 4 | Yes: 9 takes |
| Shotgun | 2 | No. Two fire takes only; borrows the rifle hold |
| Submachine gun | 3 | No. Borrows the rifle hold |
| Sniper | 3 | No. Borrows the rifle hold |
| Light machine gun | 1 | No. Borrows the rifle hold |

So every long gun is held like a rifle and maths bends the hands to fit. The game's own notes list what is still wrong:
cover is left-handed, there is no prone gun take, a punch swings the gun hand, guns are not pushed back from walls, and a
full look at every weapon hold was never done. Before October the guns floated 55-88 cm from the hands; that was fixed to
4-7 cm. Dive-and-shoot was designed but never wired in (`docs/hub/20-systems/weapons/DIVE_AND_SLIDE_WITH_GUNS.md`).

## The fix in Unreal

The standard way, which Epic's own free shooter sample uses, and every part of it is in the 5.9 install:

1. **Each weapon class gets its own animation layer**: pistol, machine pistol, submachine gun, rifle, shotgun, sniper and
   light machine gun. Swapping guns swaps the layer; the rest of the body's animation is untouched.
2. **Every gun model carries two marked points**: where the firing hand grips, and where the support hand rests. The
   game's hold standard already names these (`grip-primary`, `grip-support`, `muzzle`, `stock`).
3. **The firing hand is animated; the support hand is pulled onto its point** every frame, so it sits right on every gun
   whatever its shape.
4. **On top**: an aim layer that points the arms where the camera looks, recoil added per shot, and the gun pulled back
   when a wall is close.
5. **Diving while shooting**: the dive clip drives the body, the aim layer keeps the arms on target, and Motion Warping
   lands the dive where it was aimed.
6. **People in cars**: each character is attached to a seat point in a seated pose with hands placed on the wheel. The
   vehicle data already has seats with hand and foot points, and the browser game already shows occupants this way.

## What we have, free and owned

| Source | What it gives | Cost |
|---|---|---|
| Our "Characters" pack (in the library, not yet copied into the project) | Pistol: 29 files (aim, 8-way walk and jog, fire, reload, equip). Rifle: 39 files, plus 8 hit reactions | Owned |
| MocapOnline Pistol Pro and Rifle Pro, full sets | The repo's buy list counts 845 and 892 clips. The browser game uses 18 of them | Owned (whether the Unreal versions are in the Epic library is unchecked) |
| "Dual Gun Stylized Animation Pack" | A four-direction dive-and-shoot set, never brought into the game | Owned per the game's notes (unchecked) |
| Epic's Lyra sample | Working pistol and rifle animation layers to copy the setup from | Free |
| The engine's shooter template | Pistol and rifle animation setups with hand-adjust rigs | In the install |

**The real gap is shotgun and submachine gun clips.** Nothing owned or free covers them.

## If buying (prices as listed 2026-11-12, US dollars before tax; none lists 5.9)

| Product | Price | Covers | Note |
|---|---|---|---|
| Tactical FPS Shooter Pack (KINEMATION) | $99.99 | Pistol, revolver, submachine gun, rifles, shotgun, light machine gun, sniper; first and third person | The one pack that lists both gaps. Built first-person-first, so watch a third-person video before buying |
| Rifle Animset Pro (Kubold) | $59.99 | 120+ rifle clips including prone | Fills the missing prone take |
| Pistol Animset Pro (Kubold) | $49.99 | 150 clips with aim offsets | We already own a pistol set |

Not recommended: full shooter project templates ($110 to $180). Each brings a whole game framework to pull apart.
No pack with dive-shooting was found for sale; the owned Dual Gun set is the source for that.

Links are in the answer given to Robert and in research note 2's store section.

## Order (after the controller; with Plan 10's weapons stage)

| # | Stage | Needs Unreal? | Done when |
|---|---|---|---|
| G0 | Find and list every gun clip we own, by class, including the Dual Gun dive set; export the 16 gun models with their grip points | No | A table: class, clip count, what is missing |
| G1 | One pistol and one rifle held right on our hero: layer, grip, support hand, aim | Yes | Support hand within 2 cm of its point through a full aim sweep and while walking 8 ways; the muzzle points within 1 degree of the crosshair |
| G2 | All six classes, all 16 guns | Yes | The same two numbers for each of the 16; a line-up picture of all 16 held, for Robert's eye |
| G3 | Fire, recoil, reload, wall push-back, shooting from cover on the correct hand | Yes | Each gun fires at its table rate within 2%; the gun never passes through a wall in a 100-position test |
| G4 | Dive-and-shoot in four directions; prone shooting | Yes | Shots fired during a dive land within 1 degree of the crosshair; dive covers its rule distance within 5% |
| G5 | Shooting from a car window; occupants visible | Yes | Driver and passengers visible from outside in all 11 vehicles; hands on the wheel within 3 cm |

Each stage is checked at 30, 60, 120 and uncapped frames per second.
