# Research note 1: engine facts that settle plan choices

Read from the installed Unreal 5.9 source on 2026-11-12 (nothing was launched). Paths are under
`C:\Program Files\Epic Games\UE_5.9\Engine\`. "Status" is the plugin's own flag. UNVERIFIED means not confirmed.

| # | Question | Answer | Evidence | What the plans do with it |
|---|---|---|---|---|
| 1 | Can block buildings be meshed during play, with collision built off the main thread? | Yes. `UDynamicMeshComponent` (engine module) has `bUseAsyncCooking`, `bDeferCollisionUpdates`, `EnableComplexAsSimpleCollision`. `UProceduralMeshComponent` also works. | `Source\Runtime\GeometryFramework\Public\Components\DynamicMeshComponent.h` L714, L777, L788 | Plan 2 way A stands: Dynamic Mesh per building chunk, async collision |
| 2 | Nanite for meshes made during play? | No. "NaniteBuilder module is an editor module". | `Source\Developer\NaniteBuilder\NaniteBuilder.Build.cs` L5 | Nanite only for pre-made kit pieces (Plan 2 way C) |
| 3 | Many repeated pieces, removed one at a time when hit? | Instanced meshes support stable ids, per-instance data and cheap removal. Scene-wide limit 16.7 million instances. | `InstancedStaticMeshComponent.h` L181, L294-304; `SceneDefinitions.h` L15 | Kit pieces, furniture, props, far buildings as instances; per-instance data carries damage state |
| 4 | Can Unreal's own map streaming take content made during play? | No API found for it. UNVERIFIED whether a workaround exists. | `WorldPartition.h` L511 | Plan 1 stands: our own cell streamer |
| 5 | PCG during play? | Yes. Radii 4 m to 2,048 m per grid; 16 components generating at once; 5 ms of each frame by default. Status: normal (not Beta). | `PCGCommon.h` L604-629; `PCGRuntimeGenScheduler.cpp` L73-99; `PCGGraphExecutor.cpp` L82 | Dressing only. Its 5 ms slice goes in the frame budget |
| 6 | Crowds on roads that are generated during play? | Lane data can be registered during play (`RegisterZoneGraphData` is public, not editor-only). The normal lane builder is editor-only, but the low-level build functions are exported. Whether that works in practice: UNVERIFIED. The Mass gameplay, AI, crowd and ZoneGraph plugins: Experimental. No Mass Traffic in the engine. | `ZoneGraphSubsystem.h` L46, L160-164; `ZoneGraphBuilder.h` L102-103 | A spike before the crowd stage (W5): one cell's lanes from our lane graph. Fallback stays our own walkers |
| 7 | Menus with gamepad focus and button pictures? | CommonUI has both. Status: normal, off by default. MVVM (data binding) is Beta. CommonGame is not in the engine (ships with Epic's Lyra sample). | `CommonUIActionRouterBase.h` L129; `CommonActionWidget.h` L22 | UI plan: CommonUI for menus, our own small layer manager |
| 8 | 385 powers from data in the ability system? | Effects are fixed assets, "immutable at runtime". The supported route is a few generic abilities and effects whose numbers are passed in per use (`SetByCaller`, `SourceObject`). | `GameplayEffect.h` L31, L76; `GameplayAbilitySpec.h` L91-94, L185 | Powers plan: one generic ability per engine type (29 used by the roster); each power row rides along as data |
| 9 | Saving a damaged world? | Only raw save slots (`SaveDataToSlot`, async). The one world-state framework is Experimental and tied to streamed levels. | `GameplayStatics.h` L1092, L1104 | Our own per-cell change journal, as today |
| 10 | Walking paths on generated ground? | Yes. Navigation can build during play around chosen actors only (default radius 3,000 cm). | `NavigationData.h` L525-541; `NavigationSystem.h` L392, L577 | Only for characters that are fully real near the hero; crowds stay on lanes |
| 11 | How far from the centre before things go wrong? | Positions hold to about 44 million km. Some collision query results are still single-precision, with Epic's own note about "precision issues resulting in collision failures". That value is the distance along one query, not the distance from the centre, so whether fights fail far out is **unproven until the far-origin test runs**. Re-centring the world still exists in the engine. | `EngineDefines.h` L41, L59; `Chaos\Interface\SQTypes.h` L67, L138; `World.h` L4410 | Keep re-centring on the player as the ready fallback (the browser game already does it past 25 km). The far-origin test (W3, S5) decides |

## What changed in the plans because of this

- Plan 1 and Plan 6: re-centring the world on the player stays the ready fallback. Whether it is needed is decided by the
  far-origin test, not by this note. The browser game already re-centres past 131,072 units (25 km).
- Plan 2: way A is confirmed workable in a packaged game. Run-time Nanite is ruled out.
- Plan 1: crowds on generated roads need a spike; nothing in the engine does it out of the box.
