## OpenAI ChatGPT v1.2026.006 NPC.Tracking.Blueprint.Interface.md

**Q: I need an unreal engine blueprint interface parameter list to track all NPCs in the game world in realtime. A ThirdPerson character blueprint can then query a hybrid map/array data structure (perhaps comprised of gameplay tags, instead of object references, for runtime safety) to make gameplay decisions based on characteristics in the data structure. Here's what I have:***
```
BPI_Carousel:

Indirection: BP_NPC_* << >> BP_ThirdPerson

Parameters:

RegisterNPC
UnregisterNPC (auto UnregisterSlot)
SlotAvailable
RegisterSlot
UnregisterSlot**
```
**A:** Result: make the interface revolve around (1) a stable NPC key, (2) a compact “snapshot” payload, and (3) query functions that return *copies* of data (arrays/maps of structs), not live object references. Use `GameplayTag` / `GameplayTagContainer` for classification, and keep object references optional/soft.

BPI_Carousel (Blueprint Interface) — recommended parameter list

Types (define as Blueprint types)

* `struct FCarouselNPCKey`

  * `int32 NPCId` (assigned by registry; stable while registered)
  * `GameplayTag NPCTypeTag` (optional, if you want key namespace by type)
* `struct FCarouselNPCSnapshot`

  * `FCarouselNPCKey Key`
  * `GameplayTagContainer Traits` (faction, archetype, weapon, alert-state, etc.)
  * `GameplayTagContainer States` (alive/dead, engaged, stunned, etc.)
  * `FVector LocationWS`
  * `FRotator RotationWS`
  * `float DistanceToPlayer` (optional; can be computed client-side)
  * `bool bIsRelevant` (LOS, within radius, etc.)
  * `float Timestamp` (world seconds when produced)
  * `TSoftObjectPtr<AActor> NPCActorSoft` (optional; avoid hard refs)
  * `FGuid StableGuid` (optional; if you already have one per NPC)

Core registration (NPC → Registry)

1. `RegisterNPC`

* In: `TSoftObjectPtr<AActor> NPCActorSoft` (or `AActor` if you accept hard refs)
* In: `GameplayTagContainer Traits`
* In: `GameplayTagContainer States`
* In: `FVector LocationWS`
* In: `FRotator RotationWS`
* Out: `FCarouselNPCKey Key`
* Out: `bool bSuccess`

2. `UnregisterNPC`

* In: `FCarouselNPCKey Key`
* Out: `bool bSuccess`

3. `UpdateNPCSnapshot` (fast path, called on timer or events, not Tick for every NPC)

* In: `FCarouselNPCKey Key`
* In: `GameplayTagContainer Traits` (optional; can be empty to mean “no change”)
* In: `GameplayTagContainer States` (optional)
* In: `FVector LocationWS`
* In: `FRotator RotationWS`
* In: `bool bIsRelevant`
* Out: `bool bSuccess`

Slot system (if you truly need “carousel slots” as a second indirection)
4) `RegisterSlot`

* In: `GameplayTag SlotType` (or `GameplayTagContainer SlotFilters`)
* In: `int32 PreferredIndex` (use `-1` for any)
* Out: `int32 SlotIndex`
* Out: `bool bSuccess`

5. `UnregisterSlot`

* In: `int32 SlotIndex`
* Out: `bool bSuccess`

6. `SlotAvailable`

* In: `GameplayTag SlotType`
* Out: `bool bAvailable`
* Out: `int32 FirstFreeIndex` (or `-1`)

7. `AssignNPCToSlot` (optional but usually needed if slots exist)

* In: `int32 SlotIndex`
* In: `FCarouselNPCKey Key`
* Out: `bool bSuccess`

8. `UnassignNPCFromSlot` (or fold into `UnregisterNPC`)

* In: `int32 SlotIndex`
* Out: `bool bSuccess`

Queries (ThirdPerson → Registry)
9) `GetAllNPCKeys`

* Out: `TArray<FCarouselNPCKey> Keys`

10. `GetSnapshotByKey`

* In: `FCarouselNPCKey Key`
* Out: `FCarouselNPCSnapshot Snapshot`
* Out: `bool bFound`

11. `GetSnapshotsByTraitQuery`

* In: `GameplayTagQuery TraitQuery` (preferred over ad-hoc “must have all tags”)
* In: `GameplayTagQuery StateQuery`
* In: `float MaxDistance` (use `<=0` for ignore)
* Out: `TArray<FCarouselNPCSnapshot> Snapshots`

12. `GetNearestByTraitQuery` (common gameplay decision)

* In: `GameplayTagQuery TraitQuery`
* In: `GameplayTagQuery StateQuery`
* In: `FVector FromLocationWS`
* In: `float MaxDistance`
* Out: `FCarouselNPCSnapshot Snapshot`
* Out: `bool bFound`

13. `GetSlotOccupant`

* In: `int32 SlotIndex`
* Out: `FCarouselNPCKey Key`
* Out: `bool bOccupied`

14. `GetSlotSnapshot`

* In: `int32 SlotIndex`
* Out: `FCarouselNPCSnapshot Snapshot`
* Out: `bool bFound`

Design constraints that keep this “runtime safe”

* Do not expose `AActor` hard references in query outputs; expose `Key` + snapshot value types. If you must expose identity, use `TSoftObjectPtr` or `FGuid`.
* Keep `Traits/States` as `GameplayTagContainer`, and prefer `GameplayTagQuery` for filtering.
* Prefer event-driven updates (`UpdateNPCSnapshot` on state changes + a low-rate timer for location), not every-NPC Tick.

Minimal variant (if you want the smallest workable interface)

* `RegisterNPC(Key out, Traits, States, Location, Rotation)`
* `UnregisterNPC(Key)`
* `UpdateNPCSnapshot(Key, Traits, States, Location, Rotation)`
* `GetSnapshotsByTraitQuery(TraitQuery, StateQuery, MaxDistance)`

If you want, I can also provide the exact `USTRUCT` field list (names/types) to mirror in Blueprints and a suggested registry owner (GameState vs WorldSubsystem) for multiplayer replication.
