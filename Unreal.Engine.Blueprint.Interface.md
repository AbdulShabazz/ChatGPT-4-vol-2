## OpenAI ChatGPT v1.2026.006 Unreal.Engine.Blueprint.Interface.md

**Q: I need an unreal engine blueprint interface parameter list that interacts between animation blueprints and their owner NPC character blueprints. Can you complete the parameter list? Here is what I have:**
```
BPI_NPC_AnimNotify:
Indirection: ABP_NPC_* << >> BP_NPC_*
```
**A:** Result: a complete, minimal Blueprint Interface (BPI) surface that cleanly routes Animation Blueprint (AnimBP) notifies and queries through the owning NPC Character, without hard-coupling either side.

BPI_NPC_AnimNotify
Indirection: `ABP_NPC_*  << >>  BP_NPC_*`

Parameter conventions (apply to all):

* `NPC` (BP_NPC base type): `Object Reference` (or `Pawn/Character` base if you need polymorphism)
* `Mesh` (SkeletalMeshComponent): `Object Reference` (optional but useful for multi-mesh NPCs)
* `NotifyName` (Name): stable identifier for notify
* `EventTag` (GameplayTag or Name): optional categorization
* `Montage` (AnimMontage): optional
* `Sequence` (AnimSequenceBase): optional
* `SectionName` (Name): optional
* `NotifyTime` (Float): seconds within the source asset
* `NormalizedTime` (Float 0..1): if available
* `bIsEnd` (Bool): for window end, otherwise use dedicated End event
* `Context` (Struct): packed extras (see end)

1. Ownership / binding

1) `GetOwningNPC`

* In: `AnimInstance` (AnimInstance Object Reference) [optional]
* Out: `NPC` (BP_NPC Object Reference)
* Purpose: AnimBP resolves owner once; caller can cache.

2. `RegisterAnimInstance`

* In: `NPC`, `AnimInstance` (AnimInstance Object Reference), `Mesh`
* Out: `bAccepted` (Bool)
* Purpose: NPC stores anim instance reference(s), validates mesh, sets delegates if needed.

3. `UnregisterAnimInstance`

* In: `NPC`, `AnimInstance`, `Mesh`

2. Notify events (AnimBP -> NPC)

4) `Notify_BeginWindow`

* In: `NPC`, `NotifyName`, `EventTag`, `Duration` (Float), `NotifyTime`, `NormalizedTime`, `Montage`, `SectionName`, `Mesh`, `Context`
* Purpose: open a gameplay/UI decision window.

5. `Notify_EndWindow`

* In: `NPC`, `NotifyName`, `EventTag`, `NotifyTime`, `NormalizedTime`, `Montage`, `SectionName`, `Mesh`, `Context`

6. `Notify_Marker`

* In: `NPC`, `NotifyName`, `EventTag`, `NotifyTime`, `NormalizedTime`, `Sequence`, `Mesh`, `Context`
* Purpose: one-shot marker not tied to a window.

7. `Notify_HitboxOn`

* In: `NPC`, `NotifyName`, `EventTag`, `HitboxId` (Name), `SocketOrBone` (Name), `Mesh`, `Context`

8. `Notify_HitboxOff`

* In: `NPC`, `NotifyName`, `EventTag`, `HitboxId`, `Mesh`, `Context`

9. `Notify_Footstep`

* In: `NPC`, `Foot` (Name: "L"/"R" or socket), `SurfaceType` (PhysicalSurface Enum) [optional], `Intensity` (Float), `Mesh`, `Context`

10. `Notify_PlaySFX`

* In: `NPC`, `SFXId` (Name or GameplayTag), `SocketOrBone` (Name), `Gain` (Float), `Pitch` (Float), `Mesh`, `Context`

11. `Notify_SpawnVFX`

* In: `NPC`, `VFXId` (Name or GameplayTag), `SocketOrBone`, `Transform` (Transform), `Mesh`, `Context`

12. `Notify_GameplayCue`

* In: `NPC`, `CueTag` (GameplayTag), `Magnitude` (Float), `Mesh`, `Context`
* Purpose: single standardized hook if you use GameplayTags heavily.

3. Queries (NPC -> AnimBP) and decisions (NPC -> AnimBP)

13) `Query_CanTakeAction`

* In: `NPC`, `ActionTag` (GameplayTag), `Context`
* Out: `bCan` (Bool), `FailReason` (GameplayTag or Name)

14. `Get_CurrentState`

* In: `NPC`
* Out: `LocomotionState` (Enum), `CombatState` (Enum), `bInMontage` (Bool), `ActiveMontage` (AnimMontage)

15. `Request_PlayMontage`

* In: `NPC`, `Montage`, `PlayRate` (Float), `StartSection` (Name), `BlendIn` (Float), `BlendOut` (Float), `bInterrupt` (Bool), `Context`
* Out: `bStarted` (Bool)

16. `Request_StopMontage`

* In: `NPC`, `Montage` (optional), `BlendOut` (Float), `Context`
* Out: `bStopped` (Bool)

17. `Request_SetNextSection`

* In: `NPC`, `Montage`, `FromSection` (Name), `ToSection` (Name)
* Out: `bSet` (Bool)

18. `Request_SetAnimLayerMode`

* In: `NPC`, `Mode` (Enum: e.g., Unarmed/Armed/Alert/Investigate), `bImmediate` (Bool)
* Out: `bApplied` (Bool)
* Purpose: NPC drives linked layers / state machine switches.

19. `Push_AnimParamSet`

* In: `NPC`, `Params` (Struct: speeds, aim offsets, gait, stance, etc.)
* Out: `bApplied` (Bool)
* Purpose: if you prefer event-driven param updates vs tick polling.

4. Aim / look / IK (optional but common)

20) `Get_AimData`

* In: `NPC`
* Out: `AimWorld` (Rotator), `LookAtWorld` (Vector), `bHasTarget` (Bool)

21. `Get_IKTargets`

* In: `NPC`
* Out: `LeftHandIK` (Transform), `RightHandIK` (Transform), `bValid` (Bool)

5. Damage / reactions

22) `Notify_TakeHit`

* In: `NPC`, `HitDirection` (Vector), `HitStrength` (Float), `DamageTypeTag` (GameplayTag), `Mesh`, `Context`
* Out: `bConsumed` (Bool)

23. `Request_PlayReaction`

* In: `NPC`, `ReactionTag` (GameplayTag), `bForce` (Bool), `Context`
* Out: `bStarted` (Bool)

Context struct (recommended)
`FAnimNotifyContext`:

* `Instigator` (Actor)
* `Target` (Actor)
* `MoveId` (Name or GameplayTag)
* `VariationIndex` (Int)
* `Seed` (Int)
* `bFromServer` (Bool)
* `NetId` (Int) [optional correlation]
* `UserFloat0/1` (Float)
* `UserName0/1` (Name)
* `UserTagContainer` (GameplayTagContainer) [if using tags]

Notes (implementation constraints that avoid pain)

* Keep the interface implemented by `BP_NPC_Base` (or a shared parent), and have AnimBP call it via `TryGetPawnOwner -> DoesImplementInterface -> Execute_*`.
* Avoid passing raw `AnimNotify` class references in the interface; use `NotifyName`/`GameplayTag` to keep assets decoupled and rename-safe.

If you want this to be “minimal core” only, keep: `RegisterAnimInstance`, `Notify_BeginWindow`, `Notify_EndWindow`, `Notify_Marker`, `Query_CanTakeAction`, `Request_PlayMontage`, `Request_SetNextSection`, and `Get_CurrentState`.
