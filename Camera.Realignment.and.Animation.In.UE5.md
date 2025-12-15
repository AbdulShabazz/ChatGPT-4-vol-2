## OpenAI ChatGPT v1.2025.336 Camera.Realignment.In.UE5.md

**Q: In Unreal Engine 5.7, if after one second my third person character hasn't updated the camera lookat, I'd like the follow camera to realign toward his direction of travel!**

**A:** Implement “camera recenter after 1 s of no look input” by tracking the last non-deadzone look input timestamp, then (when moving) interpolating the controller yaw (or spring-arm yaw) toward the character’s travel direction.

Blueprint (UE 5.7 ThirdPerson template-style)

1. Variables

* `LastLookInputTime` (float)
* `RecenterDelay` = 1.0 (float)
* `RecenterSpeed` = 6.0–12.0 (float, higher = faster snap)
* `MinMoveSpeed` = 50.0 (float)

2. Update the timestamp when the player actually looks

* On your Look input (Enhanced Input `IA_Look`):

  * Break Vector2D (X=Yaw, Y=Pitch)
  * If `Abs(X) > Deadzone OR Abs(Y) > Deadzone`:

    * `LastLookInputTime = Get Game Time in Seconds`

3. Tick: recenter when idle + moving

* Event Tick:

  * `Now = Get Game Time in Seconds`
  * `bLookIdle = (Now - LastLookInputTime) > RecenterDelay`
  * `Speed = Get Velocity -> VectorLength`
  * If `bLookIdle AND Speed > MinMoveSpeed`:

    * `Dir = Get Velocity` (or `Get Actor Forward Vector` if you prefer character facing)
    * `TargetYaw = Make Rot from X(Dir) -> Break Rot -> Yaw`
    * `Current = Get Control Rotation`
    * Build `TargetRot = (Pitch = Current.Pitch, Yaw = TargetYaw, Roll = 0)`
    * `NewRot = RInterp To(Current, TargetRot, DeltaSeconds, RecenterSpeed)`
    * `Set Control Rotation(NewRot)`

Notes

* This assumes your camera follows controller rotation (typical: SpringArm “Use Pawn Control Rotation” enabled).
* Keep pitch unchanged so you don’t fight vertical aiming.
* Gate it off when aiming/lock-on/menus are active.

C++ equivalent (Character Tick)

```cpp
// In your Character.h
float LastLookInputTime = 0.f;
float RecenterDelay = 1.0f;
float RecenterSpeed = 8.0f;
float MinMoveSpeed = 50.0f;

void NotifyLookInput(float AxisX, float AxisY, float Deadzone = 0.05f)
{
    if (FMath::Abs(AxisX) > Deadzone || FMath::Abs(AxisY) > Deadzone)
        LastLookInputTime = GetWorld()->GetTimeSeconds();
}

// In Tick
void AMyChar::Tick(float DeltaSeconds)
{
    Super::Tick(DeltaSeconds);

    const float Now = GetWorld()->GetTimeSeconds();
    const FVector V = GetVelocity();
    if ((Now - LastLookInputTime) <= RecenterDelay) return;
    if (V.Size() <= MinMoveSpeed) return;

    AController* C = GetController();
    if (!C) return;

    const FRotator Current = C->GetControlRotation();
    const float TargetYaw = V.ToOrientationRotator().Yaw;

    FRotator Target(Current.Pitch, TargetYaw, 0.f);
    FRotator NewRot = FMath::RInterpTo(Current, Target, DeltaSeconds, RecenterSpeed);
    C->SetControlRotation(NewRot);
}
```

If you instead want to rotate the spring arm directly (not the controller), apply the same yaw interpolation to the spring arm’s relative/world rotation, and disable “Use Pawn Control Rotation” for that arm.

**Q: Clarification: In Unreal Engine 5.7 Blueprints, if after one second my third person character hasn't updated the camera lookat, I'd like the follow camera to realign toward his direction of travel!**

**A:** Blueprint implementation: track time since last non-deadzone “look” input; after 1.0 s of no look input and while moving, interpolate the camera rig yaw toward the character velocity heading.

Setup assumptions (Third Person template)

* Character has `CameraBoom` (SpringArmComponent) + `FollowCamera`.
* Character uses controller yaw for camera orbit (default template).

1. Variables (in Character Blueprint)

* `LastLookInputTime` (Float, default 0)
* `RecenterDelay` (Float, default 1.0)
* `RecenterInterpSpeed` (Float, default 8.0)
* `LookDeadzone` (Float, default 0.05)
* `MinMoveSpeed` (Float, default 50.0)

2. Stamp the last look time (Enhanced Input)

* `IA_Look` (Triggered) gives `Value` as Vector2D

  * `Break Vector2D` -> `LookX` (Yaw), `LookY` (Pitch)
  * Branch: `(Abs(LookX) > LookDeadzone) OR (Abs(LookY) > LookDeadzone)`

    * True: `LastLookInputTime = Get Game Time in Seconds`

3. Recenter on Tick (controller-based; simplest with template)
   Event Tick:

* `Now = Get Game Time in Seconds`
* `bLookIdle = (Now - LastLookInputTime) > RecenterDelay`
* `Speed = Get Velocity -> VectorLength`
* Branch: `bLookIdle AND (Speed > MinMoveSpeed)`

  * True:

    * `Vel = Get Velocity`
    * `VelRot = Make Rot from X (Vel)`  (or `Conv_VectorToRotator` then use Yaw)
    * `TargetYaw = Break Rotator(VelRot).Yaw`
    * `CurrentControlRot = Get Control Rotation`
    * `TargetControlRot = Make Rotator( Pitch = CurrentControlRot.Pitch, Yaw = TargetYaw, Roll = 0 )`
    * `NewControlRot = RInterp To( CurrentControlRot, TargetControlRot, DeltaSeconds, RecenterInterpSpeed )`
    * `Set Control Rotation(NewControlRot)`

Result: camera realigns toward direction of travel without changing pitch.

Common gotchas

* If your camera does not respond: ensure `CameraBoom` has `Use Pawn Control Rotation = true` (default template).
* If your character rotates with the camera but you want independent travel-facing: set Character `Use Controller Rotation Yaw = false` and in CharacterMovement enable `Orient Rotation to Movement = true` (camera still recenters via controller yaw; movement uses velocity-facing).

Alternative: rotate the spring arm instead of controller (decoupled camera)
Use this only if you disable controller-driven orbit:

* Set `CameraBoom -> Use Pawn Control Rotation = false`
* Tick logic becomes:

  * `Current = CameraBoom -> Get Relative Rotation`
  * `Target = (Pitch = Current.Pitch, Yaw = TargetYaw, Roll = 0)`
  * `New = RInterp To(Current, Target, DeltaSeconds, RecenterInterpSpeed)`
  * `CameraBoom -> Set Relative Rotation(New)`

Minimal tuning guidance

* `RecenterInterpSpeed`: 6–12 typical.
* `MinMoveSpeed`: 20–100 depending on your locomotion speeds.
* `LookDeadzone`: match your input deadzone so micro-noise doesn’t block recentering.

**Q: The camera recentering action is non-blocking: the player always retains override control over the camera.**

**A:** Make recentering non-blocking by (1) never disabling look input, (2) stopping recenter immediately when any look input is detected, and (3) blending recenter strength with current input magnitude so there is no “fight” during tiny inputs.

Blueprint pattern (controller-yaw recenter)

Variables

* `LastLookInputTime` (float)
* `RecenterDelay` = 1.0
* `RecenterRamp` = 0.25 (seconds; 0 = instant)
* `RecenterSpeed` = 8.0 (interp speed)
* `LookDeadzone` = 0.05
* `MinMoveSpeed` = 50.0

1. Look input event (Enhanced Input)

* `IA_Look (Triggered)`:

  * `Look = Value (Vector2D)`
  * If `Abs(Look.X) > LookDeadzone OR Abs(Look.Y) > LookDeadzone`:

    * `LastLookInputTime = Get Game Time in Seconds`

2. Tick: compute a recenter weight that yields to player input
   Event Tick:

* `Now = Get Game Time in Seconds`
* `Idle = Now - LastLookInputTime`
* `Speed = Get Velocity -> VectorLength`
* `LookMag = Vector2DLength(Last Look Value)` (store latest look Vector2D each input event; default (0,0))
* `InputAlpha = Clamp( (LookMag - LookDeadzone) / (1.0 - LookDeadzone), 0, 1 )`
* `DelayAlpha = Clamp( (Idle - RecenterDelay) / Max(RecenterRamp, KINDA_SMALL_NUMBER), 0, 1 )`
* `RecenterAlpha = DelayAlpha * (1 - InputAlpha)`
* Branch: `RecenterAlpha > 0 AND Speed > MinMoveSpeed`

  * `TargetYaw = Yaw( Make Rot from X( Get Velocity ) )`
  * `Current = Get Control Rotation`
  * `Target = Make Rotator( Pitch = Current.Pitch, Yaw = TargetYaw, Roll = 0 )`
  * `InterpSpeedEffective = RecenterSpeed * RecenterAlpha`
  * `New = RInterp To( Current, Target, DeltaSeconds, InterpSpeedEffective )`
  * `Set Control Rotation(New)`

Why this is non-blocking

* Any real look input updates `LastLookInputTime` and increases `InputAlpha`, which drives `RecenterAlpha` toward 0, stopping the recenter immediately (next tick) while leaving camera input fully active.
* Small incidental input noise is rejected by `LookDeadzone`.

If you want “instant yield”

* Set `RecenterRamp = 0` and keep the `InputAlpha` term; recenter will still drop to 0 as soon as input exceeds the deadzone.

**Q: These instructions are confusing. Where is this logic implemented? Why are names of Blueprints not in bold? Why not use `>>` and newlines/tabs to demarcate control flow?**

**A:** Implement it in your player character Blueprint’s Event Graph (the same place you already handle movement/look input). Concretely: **BP_ThirdPersonCharacter** (or your actual playable character Blueprint), not the camera component itself.

Prereqs (template-style)

* **BP_ThirdPersonCharacter**

  * Components: **CameraBoom** (SpringArmComponent) with `Use Pawn Control Rotation = true`, and **FollowCamera** attached (default Third Person template).

Variables (in **BP_ThirdPersonCharacter**)

* `LastLookInputTime` (Float, default 0)
* `LastLookValue` (Vector2D, default (0,0))
* `LookDeadzone` (Float, 0.05)
* `RecenterDelay` (Float, 1.0)
* `RecenterRamp` (Float, 0.25)  (0 = no ramp)
* `RecenterSpeed` (Float, 8.0)
* `MinMoveSpeed` (Float, 50.0)

1. Store look input + stamp time (Enhanced Input)
   In **BP_ThirdPersonCharacter** >> Event Graph:

* `IA_Look (Triggered)`

  > > `LastLookValue = Value (Vector2D)`
  > > `Break Vector2D (LastLookValue)` -> `LX`, `LY`
  > > `Branch (Abs(LX) > LookDeadzone OR Abs(LY) > LookDeadzone)`
  > >
  > > > True
  > > >
  > > > > `LastLookInputTime = Get Game Time in Seconds`

This guarantees “player override”: the moment real input arrives (beyond deadzone), the timer resets.

2. Event Tick recenter (non-blocking blend)
   In **BP_ThirdPersonCharacter** >> Event Graph:

* `Event Tick (DeltaSeconds)`

  > > `Now = Get Game Time in Seconds`
  > > `Idle = Now - LastLookInputTime`
  > > `Speed = VectorLength( Get Velocity )`
  > > `LookMag = Vector2DLength( LastLookValue )`
  > > `InputAlpha = Clamp( (LookMag - LookDeadzone) / (1.0 - LookDeadzone), 0, 1 )`
  > > `DelayAlpha = Clamp( (Idle - RecenterDelay) / Max(RecenterRamp, 0.0001), 0, 1 )`
  > > `RecenterAlpha = DelayAlpha * (1 - InputAlpha)`
  > > `Branch (RecenterAlpha > 0 AND Speed > MinMoveSpeed)`
  > >
  > > > True
  > > >
  > > > > `Vel = Get Velocity`
  > > > > `VelRot = Make Rot from X (Vel)`
  > > > > `TargetYaw = Break Rotator(VelRot).Yaw`
  > > > > `Current = Get Control Rotation`
  > > > > `Target = Make Rotator( Pitch = Current.Pitch, Yaw = TargetYaw, Roll = 0 )`
  > > > > `EffSpeed = RecenterSpeed * RecenterAlpha`
  > > > > `New = RInterp To(Current, Target, DeltaSeconds, EffSpeed)`
  > > > > `Set Control Rotation(New)`

Why this is non-blocking

* Player look input is always applied normally by your existing input bindings.
* Recenter only runs when (a) 1.0 s has elapsed since last real look input, and (b) input magnitude is near zero, and (c) character is moving.
* If the player touches the stick/mouse, `InputAlpha` rises and/or `LastLookInputTime` updates, driving `RecenterAlpha` to 0 on the next tick (no “fight”).

If you want the simplest non-blocking behavior (no ramp)

* Set `RecenterRamp = 0.0`.
* Keep `InputAlpha` gating as written (still yields instantly to input).

**Q: I would like to refactor Step 2 to a scalable design pattern not relying on Event Tick: Can we instead set timers for auditing/windowing, and executing events (e.g., preset timer expired + criteria met, therefore, execute event!)?**

**A:** Yes. Replace Event Tick with a timer-driven state machine: (1) arm a one-shot “audit” timer whenever look input goes idle (or movement starts), (2) when it expires, if criteria are still true, start a short repeating “recenter step” timer (active only while recentring), and (3) cancel immediately on any look input.

Scalable pattern: ActorComponent owns the timers

* Put all logic in an ActorComponent so it is reusable across pawns and camera rigs.

**BP_CameraRecenterComponent** (ActorComponent) responsibilities

* Input observation (look + optional move)
* Windowing/auditing via one-shot timer
* Recenter execution via repeating timer (only while active)

Wiring location

* **BP_ThirdPersonCharacter** >> Event Graph: forward input events to **BP_CameraRecenterComponent**
* **BP_CameraRecenterComponent** >> Event Graph: own the timers and call `Set Control Rotation` (via owning pawn’s controller) or rotate the spring arm (your choice)

---

Variables (in **BP_CameraRecenterComponent**)

* `LookDeadzone` (Float, 0.05)
* `RecenterDelay` (Float, 1.0)
* `AuditArmed` (Bool)
* `Recentering` (Bool)
* `MinMoveSpeed` (Float, 50.0)
* `RecenterSpeed` (Float, 8.0)
* `RecenterStepPeriod` (Float, 0.02)  (50 Hz; use 0.033 for 30 Hz)
* `YawToleranceDeg` (Float, 1.0)
* `LastLookValue` (Vector2D)
* Timer handles: `TH_Audit`, `TH_Recenter`

---

1. Forward inputs from the character
   In **BP_ThirdPersonCharacter** >> Event Graph:

* `IA_Look (Triggered)`

  > > `CamRecenterComp -> OnLookInput(Value)`

* (Optional) `IA_Move (Triggered)`

  > > `CamRecenterComp -> OnMoveInput(Value)`

---

2. One-shot audit timer (windowing) instead of Tick
   In **BP_CameraRecenterComponent** >> Event Graph:

Custom Event: `OnLookInput(LookValue : Vector2D)`

> > `LastLookValue = LookValue`
> > `Break Vector2D` -> `LX`, `LY`
> > `bRealLook = (Abs(LX) > LookDeadzone) OR (Abs(LY) > LookDeadzone)`
> > `Branch (bRealLook)`
> >
> > > True
> > >
> > > > `CancelRecentering()`
> > > > `ArmAuditTimer()`  (restarts the 1.0 s window)

Custom Event: `OnMoveInput(MoveValue : Vector2D)` (optional)

> > If you want “recenter after 1 s of no look since movement began”:
> > `bRealMove = Vector2DLength(MoveValue) > 0.1`
> > `Branch (bRealMove AND NOT AuditArmed AND NOT Recentering)`
> >
> > > True
> > >
> > > > `ArmAuditTimer()`

Custom Event: `ArmAuditTimer()`

> > `Clear and Invalidate Timer by Handle (TH_Audit)`
> > `AuditArmed = true`
> > `Set Timer by Event`
> >
> > > Time = `RecenterDelay`
> > > Looping = false
> > > Event = `AuditExpired`

Custom Event: `AuditExpired()`

> > `AuditArmed = false`
> > `Branch (CriteriaMet())`
> >
> > > True
> > >
> > > > `StartRecentering()`

Function: `CriteriaMet() : Bool`

> > `OwnerPawn = Get Owner (cast to Pawn)`
> > `Speed = VectorLength(OwnerPawn->GetVelocity)`
> > `LookMag = Vector2DLength(LastLookValue)`
> > return `(Speed > MinMoveSpeed) AND (LookMag <= LookDeadzone)`
> > (extend with `NOT bIsAiming`, `NOT bIsInMenu`, etc.)

---

3. Execution timer (only active while recentring)
   Custom Event: `StartRecentering()`

> > `Branch (Recentering)` -> if true return
> > `Recentering = true`
> > `Set Timer by Event`
> >
> > > Time = `RecenterStepPeriod`
> > > Looping = true
> > > Event = `RecenterStep`

Custom Event: `RecenterStep()`

> > `Branch (NOT CriteriaMet())`
> >
> > > True
> > >
> > > > `CancelRecentering()` (movement stopped or player touched look)
> > > > False
> > > > `OwnerPawn = Get Owner (cast to Pawn)`
> > > > `Vel = OwnerPawn->GetVelocity`
> > > > `TargetYaw = Yaw( Make Rot from X(Vel) )`
> > > > `Controller = OwnerPawn->GetController`
> > > > `Current = Controller->GetControlRotation`
> > > > `Target = Make Rotator( Pitch = Current.Pitch, Yaw = TargetYaw, Roll = 0 )`
> > > > `New = RInterp To( Current, Target, RecenterStepPeriod, RecenterSpeed )`
> > > > `Controller->SetControlRotation(New)`
> > > > `YawError = Abs( Delta( Current.Yaw, TargetYaw ) )`
> > > > `Branch (YawError <= YawToleranceDeg)`
> > > >
> > > > > True
> > > > >
> > > > > > `CancelRecentering()`  (optional; or keep following velocity continuously)

Custom Event: `CancelRecentering()`

> > `Clear and Invalidate Timer by Handle (TH_Recenter)`
> > `Recentering = false`

Non-blocking guarantee

* Any real look input calls `CancelRecentering()` immediately and re-arms the audit window.
* No input is consumed or disabled; you are only applying a competing rotation when the player is idle.

Notes

* This still “updates over time,” but only while recentring (the repeating timer is dormant otherwise). This is the timer-based replacement for Tick.
* If you want smoother, frame-perfect motion without Tick: use a **Timeline** inside `StartRecentering()`; the timeline’s update is effectively per-frame but self-contained and only runs while active. The repeating timer approach is usually sufficient and easier to gate/cancel.

**Q: Update to use a `timeline` inside `StartRecentering()`!**

**A:** Implement the Timeline in the owning **Actor Blueprint** (your playable character), because Blueprint **ActorComponents** cannot host Timeline nodes; Timelines are implemented as components and are not addable inside component Blueprints. ([Epic Developer Community Forums][1])

Below is a no-Event-Tick pattern using:

* a one-shot timer for the 1.0 s “no-look” window
* a Timeline for per-frame recentering while active (non-blocking; stops immediately on look input)

Where this logic lives

* **BP_ThirdPersonCharacter** (your playable character) >> Event Graph

  * Owns: timer handle + **TL_CameraRecenter** Timeline
  * Calls: `Set Control Rotation` (template-compatible)

Variables (in **BP_ThirdPersonCharacter**)

* `LastLookValue` (Vector2D, default (0,0))
* `LookDeadzone` (Float, 0.05)
* `RecenterDelay` (Float, 1.0)
* `RecenterSpeed` (Float, 8.0)
* `MinMoveSpeed` (Float, 50.0)
* `bRecentering` (Bool)
* `bRampDone` (Bool)
* `TH_Audit` (TimerHandle)

Timeline: **TL_CameraRecenter**

* Length: 1.0 s
* Looping: true
* Float track: `Alpha`

  * Key (0.0, 0.0)
  * Key (1.0, 1.0)

Control flow

1. Input capture + windowing (no Tick)
   In **BP_ThirdPersonCharacter**:

* `IA_Look (Triggered)`

  > > `LastLookValue = Value`
  > > `Break Vector2D` -> `LX`, `LY`
  > > `Branch (Abs(LX) > LookDeadzone OR Abs(LY) > LookDeadzone)`
  > >
  > > > True
  > > >
  > > > > `CancelRecentering()`
  > > > > `ArmAuditTimer()`

* `IA_Look (Completed)` and `IA_Look (Canceled)`

  > > `LastLookValue = (0,0)`
  > > (keeps input magnitude honest when the device stops sending deltas)

Custom Event: `ArmAuditTimer()`

> > `Clear and Invalidate Timer by Handle (TH_Audit)`
> > `Set Timer by Event` (Looping=false, Time=`RecenterDelay`)
> >
> > > Event = `AuditExpired`

Custom Event: `AuditExpired()`

> > `Branch (CriteriaMet())`
> >
> > > True
> > >
> > > > `StartRecentering()`

Function: `CriteriaMet() : Bool`

> > `Speed = VectorLength(GetVelocity())`
> > `LookMag = Vector2DLength(LastLookValue)`
> > return `(Speed > MinMoveSpeed) AND (LookMag <= LookDeadzone)`

2. Start/stop recentering
   Custom Event: `StartRecentering()`

> > `bRecentering = true`
> > `bRampDone = false`
> > `TL_CameraRecenter.PlayFromStart`

Custom Event: `CancelRecentering()`

> > `bRecentering = false`
> > `TL_CameraRecenter.Stop`

3. Timeline drives recenter updates (non-blocking)
   **TL_CameraRecenter** >> `Update (Alpha)`

> > `Branch (NOT bRecentering)`
> >
> > > True -> return
> > > `Branch (NOT CriteriaMet())`
> > > True -> `CancelRecentering()` return

> > `Break Vector2D(LastLookValue)` -> `LX`, `LY`
> > `Branch (Abs(LX) > LookDeadzone OR Abs(LY) > LookDeadzone)`
> >
> > > True
> > >
> > > > `CancelRecentering()`
> > > > `ArmAuditTimer()`
> > > > return

> > `RampAlpha = Alpha`
> > `Branch (bRampDone)`
> >
> > > True  -> `RampAlpha = 1.0`
> > > False -> `If (Alpha >= 0.999) bRampDone = true`

> > `Vel = GetVelocity()`
> > `TargetYaw = Yaw( Make Rot from X(Vel) )`
> > `Current = GetControlRotation()`
> > `Target = Make Rotator( Pitch=Current.Pitch, Yaw=TargetYaw, Roll=0 )`

> > `Delta = GetWorldDeltaSeconds()`
> > `EffSpeed = RecenterSpeed * RampAlpha`
> > `New = RInterp To(Current, Target, Delta, EffSpeed)`
> > `SetControlRotation(New)`

Notes

* This is “non-blocking” because any real look input immediately stops the Timeline and hands control fully back to player-driven rotation.
* The Timeline replaces Event Tick; it only “ticks” while recentring is active.
* If you want the camera to keep aligning during long movement with no look input, leave the Timeline looping (as above). If you want “one settle then stop,” disable looping and stop when yaw error is below a threshold.

[1]: https://forums.unrealengine.com/t/timelines-in-actor-components/52540?utm_source=chatgpt.com "Timelines in Actor Components! - Feedback & Requests"

**Q: Is this correct for the timeline flow control logic? There are multiple branches whose execution flow I'm unsure of, in the previous answer.**

**A:** No. In your screenshot, **TL_CameraReset.Update** is currently executing the recenter interpolation on the wrong branch (the logic is inverted), and the “Recentering == true” path does not actually perform any interpolation.

What is wrong in the current graph (top section)

1. **The first Branch is inverted**

* You have: `Recentering Flag Bool -> NOT -> Branch`
* The **True** exec path (meaning “NOT recentering”) goes to **Interpolate Follow Camera Rotation**.
* That is backwards. Interpolation must run only when `Recentering Flag Bool == true`.

2. **The “False” path (meaning “Recentering == true”) only checks criteria and maybe cancels**

* That lower path runs **Criteria Met**, then `NOT`, then Branch to **CancelRecentering**.
* It never calls **Interpolate Follow Camera Rotation**, so when recentring is active, nothing is driving yaw changes.

Correct Timeline Update flow control (minimal, scalable)
In **BP_ThirdPersonCharacter** >> **TL_CameraReset.Update**

> > `Branch(Recentering Flag Bool)`
> >
> > > True
> > >
> > > > `Branch(CriteriaMet())`
> > > >
> > > > > True
> > > > >
> > > > > > `Interpolate Follow Camera Rotation(...)`
> > > > > > False
> > > > > > `CancelRecentering()`
> > > > > > (return)
> > > > > > False
> > > > > > (return)

Notes

* Remove the `NOT` in front of the first Branch.
* Move **Criteria Met** so it is evaluated only when recentring is active (inside the True branch of `Recentering Flag Bool`).
* Do not “arm audit timer” from Timeline Update unless you have a specific reason. In most designs:

  * **ArmAuditTimer** is triggered by *look input* (and optionally movement start), not by “criteria failed while recentring”.

About your “Invalidate Follow Camera Flag Bool” Branch

* If **Interpolate Follow Camera Rotation** already computes an “invalidate” condition (e.g., player look input detected, stopped moving, etc.), keep it, but only do:

  * `Invalidate == true` >> `CancelRecentering()`
* Avoid calling **ArmAuditTimer** from there unless that invalidate specifically means “player looked” (and you are not already arming the audit timer from the look input event).

Recommended consolidation (to eliminate branch confusion)

* Put *all* gating in **CriteriaMet()**:

  * `Speed > MinMoveSpeed`
  * `Vector2DLength(LastLookValue) <= LookDeadzone`
  * optional: not aiming / not in UI / etc.
* Then **Interpolate Follow Camera Rotation** does only the yaw interpolation step (no additional cancel logic).

If you want, paste (or screenshot) the internals of **Interpolate Follow Camera Rotation** and **Criteria Met**, and I will normalize them to a single-source-of-truth gating function with the exact pins/wires.

**Q: Here are the missing function calls.**

**A:** Your **InvalidateFollowCameraRotation** function is logically correct (deadzone check → optionally stamp `LastLookInputTime` → return the same boolean), but your **TL_CameraReset.Update** flow is still wired backwards and will not recenter while `RecenteringFlagBool == true`.

**What your `InvalidateFollowCameraRotation` currently does**

* Returns `true` when `Abs(X) > LookDeadzone OR Abs(Y) > LookDeadzone`.
* When true, it also sets `LastLookInputTimeFloat = GetGameTimeInSeconds`.
* This is correct as an “is player providing look input” probe.

Critical caveat: if you feed it a latched `LastLookValue` that never resets to 0, it will keep returning `true` forever and you will never recenter. Ensure you do:

* **BP_ThirdPersonCharacter** >> `IA_Look (Completed)` and `IA_Look (Canceled)`

  > > `Set LastLookValue = (0,0)` (or (0,0,0) if you used Vector)

---

## Fix the Timeline update wiring

Right now, `TL_CameraReset.Update` runs **InvalidateFollowCameraRotation** only when `NOT RecenteringFlagBool` is true. That is inverted. The recenter interpolation must run only when **RecenteringFlagBool == true**.

Replace your top block with this exact control flow:

**BP_ThirdPersonCharacter** >> **TL_CameraReset (Update)**

> > `Branch(RecenteringFlagBool)`
> >
> > > **True** (we are recentring)
> > >
> > > > `InvalidateFollowCameraRotation(LastLookValue.X, LastLookValue.Y)`
> > > > `Branch(ReturnValue)`
> > > >
> > > > > **True** (player moved camera)
> > > > >
> > > > > > `CancelRecentering()`
> > > > > > `ArmAuditTimer()`  (optional; only if you want the 1 s window to restart immediately)
> > > > > > `Return`
> > > > > > **False** (no player look input)
> > > > > > `Branch(CriteriaMetFlagBool)` (or call your CriteriaMet function)
> > > > > >
> > > > > > > **False**
> > > > > > >
> > > > > > > > `CancelRecentering()`
> > > > > > > > `Return`
> > > > > > > > **True**
> > > > > > > > `RecenterStep` (compute travel yaw, RInterp, SetControlRotation)
> > > > > > > > **False**
> > > > > > > > `Return` (do nothing)

What to physically change in your graph

* Delete the `NOT` node feeding the first Branch (or rewire so the Branch condition is **RecenteringFlagBool** directly).
* Move the **InvalidateFollowCameraRotation** node and the **Criteria Met** checks into the **True** exec path of `Branch(RecenteringFlagBool)`.
* Ensure the interpolation (“recenter step”) executes on the path: `Recentering == true` AND `CriteriaMet == true` AND `Invalidate == false`.

---

## Recommended simplification (removes branch ambiguity)

Stop calling **InvalidateFollowCameraRotation** from the Timeline at all, and instead stamp `LastLookInputTimeFloat` only from input events:

**BP_ThirdPersonCharacter** >> `IA_Look (Triggered)`

> > `Set LastLookValue = Value`
> > `InvalidateFollowCameraRotation(Value.X, Value.Y)` (or inline the deadzone logic)
> > If true:
> >
> > > `CancelRecentering()`
> > > `ArmAuditTimer()`

Then Timeline Update becomes only:

**TL_CameraReset (Update)**

> > `Branch(RecenteringFlagBool)`
> >
> > > True
> > >
> > > > `Branch(CriteriaMet())`
> > > >
> > > > > True: `RecenterStep`
> > > > > False: `CancelRecentering()`

This is cleaner and guarantees “player override” without any per-frame polling of input.

---

If you paste (or screenshot) your **CriteriaMet** implementation and your recenter interpolation node chain (the part that actually computes/sets yaw), I can rewrite it into a single “RecenterStep” block with no redundant branches.

**Q: Provide the logic for `RecenterStep`!**

**A:** `RecenterStep` is the single “apply one camera-yaw correction” block you call from **TL_CameraReset.Update** when:

* `RecenteringFlagBool == true`
* `CriteriaMet() == true`
* `InvalidateFollowCameraRotation == false` (no player look input)

Implement it as a **Function** (pure exec) or **Custom Event** in **BP_ThirdPersonCharacter**.

RecenterStep (controller-yaw, Third Person template compatible)
**BP_ThirdPersonCharacter** >> `RecenterStep` (Function or Custom Event)

> > `Vel = GetVelocity`
> > `Vel2D = Vector(Vel.X, Vel.Y, 0)`
> > `Speed2D = VectorLength(Vel2D)`
> > `Branch(Speed2D > MinMoveSpeed)`
> >
> > > False
> > >
> > > > `CancelRecentering()` (optional)
> > > > Return

> > > True
> > >
> > > > `TargetRot = Make Rot from X (Vel2D)`
> > > > `TargetYaw = Break Rotator(TargetRot).Yaw`

> > > > `Controller = GetController`
> > > > `Branch(IsValid(Controller))`
> > > >
> > > > > False: Return

> > > > > True
> > > > >
> > > > > > `Current = Controller -> GetControlRotation`
> > > > > > `RampAlpha = TL_CameraReset.Alpha` (float track 0→1; or 1.0 if you removed ramp)
> > > > > > `EffSpeed = RecenterSpeed * RampAlpha`

> > > > > > `Target = Make Rotator( Pitch = Current.Pitch, Yaw = TargetYaw, Roll = 0 )`
> > > > > > `Delta = GetWorldDeltaSeconds`
> > > > > > `New = RInterp To( Current, Target, Delta, EffSpeed )`
> > > > > > `Controller -> SetControlRotation(New)`

Optional “stop when aligned” (prevents endless loop when you want a one-shot settle)
Add after `SetControlRotation`:

> > `DeltaRot = Normalized Delta Rotator( A = New, B = Target )`
> > `YawErr = Abs( Break Rotator(DeltaRot).Yaw )`
> > `Branch(YawErr <= YawToleranceDeg)`
> >
> > > True
> > >
> > > > `CancelRecentering()`

Notes

* Flattening Z (`Vel2D`) prevents pitching the camera when running uphill/downhill.
* Keeping pitch from `Current.Pitch` preserves the player’s vertical aim.
* If you want “continuous follow while moving and no look input,” omit the YawTolerance stop and keep the Timeline looping.

Alternative: rotate the Spring Arm Component (SAC) instead of controller
Use only if your **Spring Arm Component (SAC)** is not using pawn control rotation (`Use Pawn Control Rotation = false`):

* Replace controller rotation reads/writes with:

  * `Current = CameraBoom -> GetRelativeRotation`
  * `CameraBoom -> SetRelativeRotation(New)`
    Everything else stays identical (use yaw only, preserve pitch).

If you show your current “set control rotation” vs “rotate spring arm” configuration (the relevant component flags), I can lock the block to the correct variant with exact node names/pins for your setup.

**Q: It's much easier to fire the delay/reset logic off of `ia_look.completed` and `ia_move.completed` that way there is no continuous polling during the input action.**

**A:** Yes. You can make the 1 s window fully event-driven off `IA_Look.Completed` (and optionally `IA_Move.Started/Completed`) so there is no continuous “am I idle?” polling while the input action is active. The only recurring update is the Timeline while recentring (which is the point).

Event-driven pattern (Blueprint, Enhanced Input)

State

* `RecenteringFlagBool`
* `TH_Audit` (TimerHandle)
* `LastLookValue` (Vector2D)

Constants

* `RecenterDelay = 1.0`
* `LookDeadzone`
* `MinMoveSpeed`

1. Look input drives cancel + window arming

**BP_ThirdPersonCharacter** >> `IA_Look.Triggered`

> > `LastLookValue = Value`
> > `bRealLook = (Abs(Value.X) > LookDeadzone) OR (Abs(Value.Y) > LookDeadzone)`
> > `Branch(bRealLook)`
> >
> > > True
> > >
> > > > `CancelRecentering()`
> > > > `ClearAndInvalidateTimer(TH_Audit)`  (optional; prevents a pending audit firing mid-look)

**BP_ThirdPersonCharacter** >> `IA_Look.Completed`

> > `LastLookValue = (0,0)`  (critical; avoids stale “look still active”)
> > `ArmAuditTimer()`

**BP_ThirdPersonCharacter** >> `IA_Look.Canceled`

> > same as Completed (treat cancel as “look ended”)
> > `LastLookValue = (0,0)`
> > `ArmAuditTimer()`

`ArmAuditTimer()`

> > `ClearAndInvalidateTimer(TH_Audit)`
> > `SetTimerByEvent(Time=RecenterDelay, Looping=false)`
> >
> > > Event = `AuditExpired`

`AuditExpired`

> > `Branch(CriteriaMet())`
> >
> > > True: `StartRecentering()`  (plays Timeline)
> > > False: return

2. Movement input (optional but recommended)

Goal: if the player starts moving after having stopped looking, you still want the 1 s window to start (otherwise you’d wait until the next Look.Completed, which may never occur if they never looked).

**BP_ThirdPersonCharacter** >> `IA_Move.Started`

> > `Branch( NOT RecenteringFlagBool )`
> >
> > > True
> > >
> > > > `ArmAuditTimer()`  (starts “no-look” window at movement onset)

**BP_ThirdPersonCharacter** >> `IA_Move.Completed`

> > `ClearAndInvalidateTimer(TH_Audit)`
> > `CancelRecentering()`  (optional; prevents yaw drift when stopping)

3. Timeline only does recenter work while active

**TL_CameraReset.Update**

> > `Branch(RecenteringFlagBool)`
> >
> > > True
> > >
> > > > `Branch(CriteriaMet())`
> > > >
> > > > > True: `RecenterStep()`  (your yaw interpolation)
> > > > > False: `CancelRecentering()`
> > > > > False: return

`CriteriaMet()` should include:

* speed check: `VectorLengthXY(GetVelocity()) > MinMoveSpeed`
* “no look input” check: `Vector2DLength(LastLookValue) <= LookDeadzone` (if you keep LastLookValue current; otherwise omit since Look.Completed gates the timer)

Key advantage of this design

* No per-frame polling to decide when to start the 1 s delay; the delay is armed only on discrete input lifecycle events.
* Player override remains immediate because `IA_Look.Triggered` cancels recenter on first real look delta.

Edge cases to account for

* If your `IA_Look.Completed` does not fire reliably for mouse input (depends on triggers/modifiers), keep the `LastLookValue = (0,0)` on Completed/Canceled and ensure the action isn’t “always triggered.” If it is, switch to using a Deadzone modifier and/or a trigger that can complete when input returns to zero.

**Q: Instead of `timeline` which cannot be used inside an actor component, can I use `rinterpto` and package my logic in a component?**

**A:** Yes. `RInterpTo` (Rotator Interpolate To) is sufficient, and you can package the entire system in an **ActorComponent** by driving `RInterpTo` from a repeating **timer** (active only while recentring). This avoids Timelines and avoids continuous polling while the player is actively providing input.

Component-based design (no Timeline, no always-on Tick)

Where it lives

* **BP_CameraRecenterComponent** (ActorComponent) attached to **BP_ThirdPersonCharacter** (or any Pawn)

What runs continuously

* Nothing, unless recentring is active.
* While recentring: a repeating timer fires `RecenterStep()` at a fixed period (e.g., 0.02 s).

---

## **BP_CameraRecenterComponent** variables

Constants

* `LookDeadzone` (Float, 0.05)
* `RecenterDelay` (Float, 1.0)
* `RecenterStepPeriod` (Float, 0.02)  (50 Hz)
* `RecenterSpeed` (Float, 8.0)
* `MinMoveSpeed` (Float, 50.0)
* `YawToleranceDeg` (Float, 1.0) (optional stop threshold)

State

* `LastLookValue` (Vector2D, (0,0))
* `bRecentering` (Bool, false)
* `TH_Audit` (TimerHandle)
* `TH_Recenter` (TimerHandle)

---

## Input hooks (called by the owning character)

In **BP_ThirdPersonCharacter** (Event Graph):

* `IA_Look.Triggered`

  > > `CamRecenterComp >> OnLookTriggered(Value)`
* `IA_Look.Completed` and `IA_Look.Canceled`

  > > `CamRecenterComp >> OnLookCompleted()`
* Optional:

  * `IA_Move.Started` >> `CamRecenterComp >> OnMoveStarted()`
  * `IA_Move.Completed` >> `CamRecenterComp >> OnMoveCompleted()`

---

## Component events

**BP_CameraRecenterComponent** >> Event Graph:

### `OnLookTriggered(LookValue : Vector2D)`

> > `LastLookValue = LookValue`
> > `bRealLook = (Abs(LookValue.X) > LookDeadzone) OR (Abs(LookValue.Y) > LookDeadzone)`
> > `Branch(bRealLook)`
> >
> > > True
> > >
> > > > `StopRecentering()`
> > > > `Clear and Invalidate Timer by Handle (TH_Audit)`  (optional; prevents an audit firing mid-look)

### `OnLookCompleted()`

> > `LastLookValue = (0,0)`
> > `ArmAuditTimer()`

### `OnMoveStarted()` (optional)

> > `Branch(NOT bRecentering)`
> >
> > > True: `ArmAuditTimer()`
> > > (Starts the 1 s window even if the player never touched look.)

### `OnMoveCompleted()` (optional)

> > `StopRecentering()`
> > `Clear and Invalidate Timer by Handle (TH_Audit)`

---

## Audit (1-second window)

### `ArmAuditTimer()`

> > `Clear and Invalidate Timer by Handle (TH_Audit)`
> > `Set Timer by Event`
> >
> > > `Time = RecenterDelay`, `Looping = false`
> > > Event = `AuditExpired`

### `AuditExpired()`

> > `Branch(CriteriaMet())`
> >
> > > True: `StartRecentering()`

### `CriteriaMet() : Bool`

> > `OwnerPawn = GetOwner (Cast to Pawn)`
> > `Speed2D = VectorLength( Vector(OwnerPawn.Velocity.X, OwnerPawn.Velocity.Y, 0) )`
> > `LookMag = Vector2DLength(LastLookValue)`
> > return `(Speed2D > MinMoveSpeed) AND (LookMag <= LookDeadzone)`

---

## Recenter execution (timer-driven `RInterpTo`)

### `StartRecentering()`

> > `Branch(bRecentering)` -> True: return
> > `bRecentering = true`
> > `Set Timer by Event`
> >
> > > `Time = RecenterStepPeriod`, `Looping = true`
> > > Event = `RecenterStep`

### `StopRecentering()`

> > `bRecentering = false`
> > `Clear and Invalidate Timer by Handle (TH_Recenter)`

### `RecenterStep()`

> > `Branch(NOT CriteriaMet())`
> >
> > > True: `StopRecentering()` >> return

> > `OwnerPawn = GetOwner (Cast to Pawn)`
> > `Controller = OwnerPawn -> GetController`
> > `Branch(IsValid(Controller))`
> >
> > > False: `StopRecentering()` >> return

> > `Vel = OwnerPawn->GetVelocity`
> > `Vel2D = Vector(Vel.X, Vel.Y, 0)`
> > `TargetYaw = Yaw( Make Rot from X(Vel2D) )`

> > `Current = Controller->GetControlRotation`
> > `Target = Make Rotator( Pitch=Current.Pitch, Yaw=TargetYaw, Roll=0 )`

> > `New = RInterp To( Current, Target, RecenterStepPeriod, RecenterSpeed )`
> > `Controller->SetControlRotation(New)`

Optional stop threshold (one-shot settle):

> > `DeltaRot = Normalized Delta Rotator( New, Target )`
> > `YawErr = Abs(DeltaRot.Yaw)`
> > `Branch(YawErr <= YawToleranceDeg)`
> >
> > > True: `StopRecentering()`

---

## Non-blocking guarantee

* Any real look delta calls `OnLookTriggered` and immediately `StopRecentering()`.
* Recenter only “runs” during the repeating timer, and that timer is only active while recentring.

If you want, specify whether you are rotating **controller yaw** (template) or **spring arm relative rotation** (decoupled camera). The `RecenterStep()` block changes slightly depending on that choice.

**Q: My component relies on a custom timer delay handle which only fires on `ia_look.completed and .cancaled`  and `ia_move.completed and .canceled`.**

**A:** That wiring is internally consistent, but it will not satisfy the original behavior unless the player has previously generated a “look action” that can later complete/cancel.

Failure mode with “Completed/Canceled only”

* If the player starts moving without ever touching the look input, `IA_Look.Completed/Canceled` will not fire, so your 1 s audit timer never arms.
* If you only arm on `IA_Move.Completed/Canceled`, you are arming when movement stops, so you cannot recenter “toward direction of travel” while traveling.

If you accept “recentering can only begin after the player has looked at least once, and only after they stop looking,” then your approach is fine. Otherwise you need one additional arming event tied to movement onset.

Component event-driven state machine (no continuous polling during the input action)

State

* `bLookActive` (bool)
* `bMoveActive` (bool)
* `LastLookValue` (Vector2D)
* `TH_Audit` (TimerHandle)
* `TH_Recenter` (TimerHandle)
* `bRecentering` (bool)

Look

* `IA_Look.Triggered(Value)`

  > > `bLookActive = (Abs(Value.X) > Deadzone OR Abs(Value.Y) > Deadzone)`
  > > if `bLookActive`:

  * `LastLookValue = Value`
  * `StopRecentering()`
  * `ClearTimer(TH_Audit)` (optional but recommended)
* `IA_Look.Completed` or `IA_Look.Canceled`

  > > `bLookActive = false`
  > > `LastLookValue = (0,0)`
  > > if `bMoveActive`:

  * `ArmAuditTimer()`   (this is your current design and it is correct)

Move

* `IA_Move.Triggered/Started(Value)`  (this is the missing piece if you want recenter without any look input)

  > > `bMoveActive = (Vector2DLength(Value) > MoveDeadzone)`
  > > if `bMoveActive AND NOT bLookActive`:

  * `ArmAuditTimer()`
* `IA_Move.Completed` or `IA_Move.Canceled`

  > > `bMoveActive = false`
  > > `ClearTimer(TH_Audit)`
  > > `StopRecentering()`

Audit → recenter execution (component timer-driven `RInterpTo`)

* `ArmAuditTimer()`

  > > `ClearTimer(TH_Audit)`
  > > `SetTimer(TH_Audit, RecenterDelay, false, AuditExpired)`
* `AuditExpired()`

  > > if `bMoveActive AND NOT bLookActive AND Speed2D > MinMoveSpeed`:

  * `StartRecentering()` (starts repeating `TH_Recenter`)
* `RecenterStep()` (repeating timer)

  > > if criteria fail: `StopRecentering()`
  > > else: compute `TargetYaw` from velocity heading, `RInterpTo(Current, Target, StepPeriod, RecenterSpeed)`, `SetControlRotation(New)` (or spring arm rotation)

If you want to keep “Completed/Canceled only” strictly

* You must accept that recentering will not start unless a look action completes/cancels while the pawn is moving (or you add some other arming event such as BeginPlay, movement mode change, or first movement input).
