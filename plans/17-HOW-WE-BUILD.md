# Plan 17: How we build it, and what to prompt

Written 2026-11-12. Robert shared a video of someone building a small pickleball game in Unreal 5.9 with Claude in under
a day, and asked: since we are not generating our models the way that video does, how do we get all of ours done, and
what should we be prompting?

## What that video's method is

| Step in the video | What it is |
|---|---|
| A 5.9 skill | A file that teaches the AI the engine's new tools, made by having it study the docs |
| Premise | A filled-in template: devices, tone, camera, what the game is |
| Four mock-ups at once | Pictures of the game being played, in four different looks; pick one |
| Grey-box gym | The game built from plain boxes to prove it feels right before any art |
| Assets on a separate track | Characters and environment made in parallel with the gameplay |
| Iterate in small steps | Minutes between changes, the person steering |
| Menus, controller pictures, sound, props | Polish passes, one prompt each |
| The AI tests it itself | Through Unreal's built-in door for AI, pressing buttons and checking the screen |

His game is one court, two characters and a ball. It cost him about $420 of AI use. Ours is far bigger, so the method
carries over but the "one day" does not.

## What we do the same

- **A 5.9 skill.** We have the research (the field guide and note 1). It is now packed as a project skill, so every
  session starts knowing the engine's AI door, the traps, and our rules.
- **Grey box first.** Our gym is the training area and obstacle course (Plans 5 and 8): plain boxes, real rules.
- **Mock-ups before looks.** Before any screen or effect is built, four pictures in parallel, and Robert picks. We make
  them on our own Cupcake box through Directors Palette, in the COF Tactics look.
- **Two tracks at once.** Rules and tests on one track (no Unreal). Art, animation and sound on another. They meet in
  the gym.
- **The AI tests it itself.** Already proven on our project: starting the game inside the editor, reading frame times,
  taking pictures, pressing inputs.
- **Small steps, Robert steering.** One ring or one station at a time, each ending in something he can play.

## What we do differently, and why

| The video | Us | Why |
|---|---|---|
| Generates characters from a picture with a paid service | Import our own blocky hero, its 624 clips and costume system | It is the game's look and it is already made |
| Generates the environment by hand in Blender | The city and houses come from our generators | 1,109 cities cannot be modelled |
| Tunes by feel only | Every rule has a number and a test before it is on screen | It is why the slice plays the same at every frame rate, and why server play will work |
| One prompt, big result | One stage, measured result | A game this size goes wrong quietly without pass lines |
| Ships to web, phone and tablet at once | PC first | The web target is experimental in 5.9; phones are a later call |
| No server | Server and client from the first ring | Robert's ruling |

## The loop for every piece

1. **Name it.** A ring, station or stage from the plans (for example "ring 1" or "station 13").
2. **Rules and tests.** Done with Unreal closed. Ends with a test count.
3. **Grey box.** On screen in the gym with plain shapes.
4. **The course sheet.** Pass or fail with numbers, at four frame rates, and under simulated lag.
5. **Robert plays.** Thumbs up or down per station, one sentence if down.
6. **Tune.** Numbers change; the sheet runs again.
7. **Dress it.** Animation, effects, sound, display. Mock-ups first where it is a look.
8. **Critic, then done.** One line to Robert: done, here is the proof.

## What to prompt

The plans hold the detail, so prompts can be short. Name the piece and say how far to take it.

### To start work

| Say | What happens |
|---|---|
| "Start the groundwork." | Phase 0: export the game's data, record every knock-back, settle the speed ladder. No Unreal, bar one short run |
| "Do ring 1, rules only." | The knock-back rules and their tests, Unreal closed |
| "Ring 1 into the gym." | Builds the grey-box stations for it, runs the course, shows the sheet |
| "Build the training area: punching bag, force pads, material walls." | The first showpiece |
| "Do the war game paper stages." | Plan 16's X0 to X5, alongside the controller |

### To see something

| Say | What happens |
|---|---|
| "Show me." | A picture or short clip of the latest thing, from the fixed cameras |
| "Run the course." | The full pass/fail sheet |
| "What's failing?" | Only the red rows, with the number that missed |
| "Launch it." | Opens the game for you to play |

### To steer after playing

Say it per station, in your own words. This shape works best:

> Station 6: thumbs down. They don't fly far enough. Double it.
> Swoop: thumbs up, don't touch it.
> Camera in the canyon fights me on the left turns.

Or just a number: "Knock-back dial to 1.5." "Walk speed down a bit." "Door too tight."

### To decide a look

| Say | What happens |
|---|---|
| "Mock up the on-screen display, four ways." | Four pictures of the game being played, COF Tactics look, made in parallel. You pick or mix |
| "Mock up the beam." (or blast, sonic boom, landing) | Same, for an effect |
| "Go with the second one, but the bars from the fourth." | It is built to match |

### To add or change scope

| Say | What happens |
|---|---|
| "Plan X." (boats, the airport, underwater, missions) | A plan like the others: what exists, how in Unreal, stages with pass lines, your calls |
| "Is this possible: ..." | A straight answer with what is proven and what is new |
| "Find me something already made for ..." | What we own, what is free, what is sold, with prices |
| "Rule: ..." | Written into the plans as a ruling, with the date |

### Words that change how I work

| Say | Means |
|---|---|
| "Plan only" | No building, no Unreal. Where we are today |
| "Unreal's free" | I can open the editor |
| "Keep going" | Next stage in the roadmap without asking |
| "Stop at the sheet" | Build and test, then wait for you to play |

## What only Robert does

- Play it and say what feels wrong. No test replaces that.
- Rule on the open calls (listed in the roadmap in the order they matter).
- Buy things: store purchases, the store account, any paid service.
- Say when Unreal can run.

## What this costs

The video's small game used about $420 of AI. No honest figure exists for ours yet. The first measured number will be
the groundwork and ring 1; after that each ring can be estimated from the last.
