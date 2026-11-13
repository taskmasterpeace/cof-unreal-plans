# Research note 2: existing character controllers, and whether to buy

Written 2026-11-12. Robert: "find some already made character controllers... do we need to buy them?"
Three searches: what we own, what is free, what is sold. Nothing was bought, downloaded or cloned.

## Answer

**No, we do not need to buy a controller.** We own one working flight controller, the free material covers ground
movement and climbing better than anything sold, and the parts nobody sells well (superhero flight, web-style swinging,
carry and throw at superhero scale, knock-back through walls) are the parts we already have rules for.

A bought framework would bring its own character and its own state machine. Our flight, boost gears, swoop and punch
rules would have to be rewritten inside it. Most are Blueprint-only, which cannot be tested the way our rules are, and
none lists Unreal 5.9.

## What we already own

| Pack | What it is |
|---|---|
| **Superhero Flight** | A real, working flight controller built on Unreal's standard character body: hover and fast flight, five flight styles, take-off, superhero landing with effects by surface (concrete, grass, sand, water), flight dodges, an aiming mode, camera field-of-view and lag blending, sonic boom and trail effects. Looks built for multiplayer (unconfirmed). All Blueprint; made for Unreal 4.25 to 5.3 |
| Unleashed Boxer, Knockdowns, Suplex, Werewolf | Clips: rolls, dodges, hit reactions in four directions, 26 knockdowns, get-ups |
| Throwing Pack | About 50 throw clips (one hand, two hands, overhead, slam). Not yet copied into our project |
| Kickboxing, Kung Fu, Dual Sword, Grenade, others | Clips, some with a basic demo character |

Not in anything we own: grappling hook, swinging, climbing, mantle, vault, ledge, wall run, ragdoll, carrying, human swimming.

**Use of the flight pack:** a parts donor and a reference. Its landing-by-surface effects, flight dodges, camera blending
and sonic-boom effects are worth lifting into our controller. Its logic stays out, because ours is tested and the same at
every frame rate.

## Free, worth using

| # | What | Licence | What it gives us | Verdict |
|---|---|---|---|---|
| 1 | Epic's Game Animation Sample (separate free download; latest is for 5.8) | Unreal projects only | Motion matching for ground movement, 500+ animations, simple vault and climb, powered ragdoll, shove and tackle | **Use the animations and setup.** Reference for the logic |
| 2 | PredictedMovement (open source, updated Oct 2026, for 5.7 with 5.8 prep) | MIT | The right pattern for boosts, slow-downs, stamina and knock-back that stays correct in online play | **Reuse the pattern** when online play is planned |
| 3 | ALS-Refactored (open source, 5.7) | MIT | Blended locomotion, mantle, roll, ragdoll get-up, in C++ | Reference only |
| 4 | Mover examples (in the engine; Experimental) | Engine licence | Epic's own flying and swimming modes, and a zipline | Reference only |
| 5 | Third Person template variants (in the engine) | Engine licence | Dash, wall jump, combat hit reactions in C++ | Reference only |

Found nothing usable and free for: superhero flight, grapple and swing, carry and throw. The best open grappling hook
project has 3 stars.

Rough hours saved (estimates, not measured): the Game Animation Sample 80+ (animation we could not make ourselves),
PredictedMovement 20-30, ALS-Refactored as a reference 15-25.

## Paid, only if wanted

Prices in US dollars before tax, as listed on Epic's store on 2026-11-12; they change, and the price tier depends on
studio revenue. No listing names 5.9.

| Product | Price | What it would save | Catch |
|---|---|---|---|
| Action-Adventure Movement System V2 | $99.99 | Vault, mantle, ledge climb, ladders, grapple hook, rope and pole swing, swimming, glider, 475+ animations. Rated 5.0 from about 70 | Blueprint only. Use for its clips and as a worked example, not as our controller |
| GrabNet | $149.99 | Grab, carry, throw and shove of objects and ragdolls that works online; C++ | No ratings yet. Only pays off if the game is online |
| Active Ragdoll System C++ v2 | $19.99 | Physical knock-back and recovery | One rating. A free alternative exists (ProcHitReact) |
| Ultimate Hit Reaction System | $69.99 | 168 hit, knockdown and launch reactions | Blueprint |

Looked at and not recommended: full combat frameworks ($275 to $700) and a replacement movement component (from about
$600). Each replaces the thing we already built.

## Recommendation

1. Buy nothing now.
2. When Unreal can run again: download Epic's free Game Animation Sample into a copy and try it on 5.9.
3. Copy the Throwing Pack into our project (we own it; it was left behind).
4. Lift the landing effects, flight dodges and camera blending out of our Superhero Flight pack.
5. If climbing and swinging animation turns out to be the gap once the grappling hook is in, the $99.99 pack is the
   one purchase most likely to pay for itself. That is a decision for then.

## Not verified

- Whether any of these open or run on the 5.9 Preview.
- Store price tiers and exact engine-version lists (the store pages did not show them).
- That the Superhero Flight pack's multiplayer actually works.
