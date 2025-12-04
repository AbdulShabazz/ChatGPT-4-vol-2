## OpenAI ChatGPT v1.2025.329 USD.Files.Support.Behavior.md

**Q: CD Projekt Red used USD files to cache parametric animations instead of VDBs for The Witcher 4, Unreal Engine demo. Do USD files natively support this behavior?**

**A:** Short answer:
Yes. USD can natively act as an animation cache container, but it does so by storing time-sampled/baked values, not by “running” parametric functions inside the file. The “parametric” part lives in the tools that generate or read the USD, not in USD’s core semantics.

Details:

1. What USD supports natively

   * USD is a scene description format with first-class support for time-varying data on attributes (positions, transforms, material parameters, etc.) via TimeSamples and timeCodes. These are literally per-time key/value samples that define an animation curve.([openusd.org][1])
   * It also supports:

     * Transform and point/vertex animation via time-sampled attributes.([openusd.org][2])
     * Higher-level animation constructs (skeletal animation, blendshapes, etc.) through schemas built on those time-sampled attributes.([NVIDIA Omniverse Docs][3])
     * Value Clips, which let you split large animated caches into per-frame/per-chunk USD files and stitch them into a single logical animated sequence. This is explicitly intended for “cache-like” workflows.([openusd.org][4])

   In film/VFX pipelines, this is exactly how a “USD cache” is used: simulations or parametric rigs produce baked geometry or transform data, which then gets written into USD as time-sampled attributes and possibly value clips.

2. What USD does *not* do by itself

   * USD does not natively evaluate parametric expressions like “x(t) = sin(ωt) * A” at runtime. It stores:

     * Either the *baked result* as discrete time samples (the usual case for caches), or
     * Custom attributes/metadata that *your* runtime can interpret as parameters and formulas.
   * In other words, the parametric system is external; USD is the carrier for the data (baked samples or parameters), not the execution engine.

3. How this differs from VDB

   * OpenVDB is a sparse volumetric data format, used for grids like density, temperature, velocity, etc. It is geared toward simulation/volume caches.
   * USD is a general scene graph / composition format. It can *reference* volumetric data (e.g., USD Volume schema that points to VDB files), but its native strength is describing geometry, transforms, materials, and their animation over time.([Foundry Learn][5])
   * So “using USD files instead of VDBs to cache parametric animations” for Witcher-style foliage/character/rig animation basically means:

     * Use USD’s time-sampled attributes / value clips to store baked transform/vertex data,
     * Rather than storing a volumetric simulation grid as OpenVDB.

4. Re: Unreal Engine + Witcher 4

   * Unreal’s USD support explicitly treats USD as an interchange format for geometry and animation, including baked animation caches (e.g., for grooms, meshes, etc.).([Epic Games Developers][6])
   * CDPR’s “cache parametric animation to USD” phrasing is consistent with:

     * Drive animation parametrically in their authoring tools,
     * Bake the resulting transforms/vertex positions to USD time samples / value clips,
     * Stream those USD caches into Unreal at runtime.

So:
USD does natively support *the caching behavior* (time-varying animation data, clip sequencing, large caches). It does not natively encode or execute parametric formulas; those are generated and consumed by the surrounding tools, with USD used as the storage and interchange layer.
```js
[1]: https://openusd.org/docs/?utm_source=chatgpt.com "Introduction to USD"
[2]: https://openusd.org/22.08/tut_xforms.html?utm_source=chatgpt.com "Transformations, Time-sampled Animation, and Layer Offsets"
[3]: https://docs.omniverse.nvidia.com/kit/docs/pxr-usd-api/105.0.2/pxr/Usd.html?utm_source=chatgpt.com "Usd module — pxr-usd-api 105.0.2 documentation"
[4]: https://openusd.org/docs/api/_usd__page__value_clips.html?utm_source=chatgpt.com "Sequencable, Re-timable Animated \"Value Clips\""
[5]: https://learn.foundry.com/katana/8.0v1/Content/ug/usd/what_is_usd.html?utm_source=chatgpt.com "What is USD?"
[6]: https://dev.epicgames.com/documentation/en-us/unreal-engine/universal-scene-description-in-unreal-engine?utm_source=chatgpt.com "Universal Scene Description in Unreal Engine"
```
**Q: Is it recommended to employ USDs over VDBs for purposes like this, such as for performance reasons?**

**A:** No. It is not generally “USD vs VDB for performance.” For the kind of parametric animation caching you’re describing (meshes, rigs, grooms in Witcher 4’s UE demo), USD is the appropriate tool; VDB is the wrong abstraction, not just a slower one.

Key points:

1. Different problem domains

   * USD: scene description + geometry + time-sampled attributes (vertex positions, transforms, blendshapes, skeletal anim, etc.). Ideal for mesh- and rig-based animation caches.
   * OpenVDB: sparse 3D grids (density, velocity, temperature, etc.). Ideal for volumes like smoke, clouds, explosions, fog.
     Using VDB to store character or foliage motion would mean encoding surfaces as volumetric grids and then somehow reconstructing geometry or rendering as volumes, which is structurally mismatched and almost always more expensive.

2. Runtime performance in Unreal Engine
   For Witcher-style parametric animation caches (cloth, foliage, grooms, deformation):

   * USD cache path:

     * Authoring: parametric rigs/sims → bake to time-sampled vertex or transform data in USD (or Alembic-like).
     * Runtime: import/stream as standard UE meshes (skeletal/geometry cache), using the engine’s native GPU triangle pipelines, LODs, culling, instancing.
       => Good cacheability, good I/O locality, straightforward GPU path, predictable cost.
   * VDB path:

     * Would require volume sampling / raymarching, or converting back to meshes at load time.
     * Much heavier memory bandwidth and ALU per pixel/voxel, and no direct use of UE’s skeletal/mesh animation optimizations.
       => Usually worse performance and complexity for anything that is fundamentally a surface, not a volume.

3. Data size and streaming

   * Mesh/transform caches in USD:

     * Data scales roughly with vertices × frames × components; can be chunked via USD Value Clips and LODed or partitioned spatially/temporally.
   * Volume grids in VDB:

     * Data scales with grid resolution; even with sparsity, high-res animated grids are large.
     * Streaming animated VDBs for real-time gameplay is intensive and best reserved for a few “hero” volume effects, not pervasive character/world animation.
       For anything that will ultimately be rendered as opaque/alpha-tested surfaces, USD/mesh caches will almost always win on I/O and memory.

4. When VDB is actually appropriate
   Prefer VDB (or NanoVDB / OpenVDB variants) when:

   * The effect is inherently volumetric (smoke, mist, volumetric clouds, magical fog, explosions).
   * You want volumetric lighting/scattering and raymarching effects.
     Even then, the common pattern is: VDB for the volume, USD for the rest of the scene, with USD Volume prims referencing VDB grids.

5. Practical recommendation

   * For “cached parametric animation” of characters, foliage, cloth, grooms, destruction meshes, etc.:

     * Use USD (or Alembic) as your cache/interchange format and bring that into Unreal as mesh/animation assets.
   * For volumetric FX:

     * Use VDB and, if needed, reference it from USD or import via UE’s volume tools.

So it is not that “USD is faster than VDB for the same job”; the jobs are different. For the Witcher 4-style UE demo use case, USD-style mesh/transform caches are the recommended and performant choice; VDB should be reserved for true volumetric content.

**Q: Can USDs be instanced (multiple Reads or Reader threads) per batch of cached animations to provide the illusion of multiple  characters?**

**A:** Yes. A single USD animation cache can be instanced many times to create the illusion of multiple characters; you do not need separate files or separate “reader threads” per character. Instancing is a first-class USD concept, and threading is mostly an implementation detail of the USD runtime.

Key pieces:

1. Instancing the cached animation
   At the USD level you typically do:

   * Create one “prototype” character prim with its baked animation (time-sampled transforms / points, possibly via value clips).([openusd.org][1])
   * In a separate shot/crowd layer, create multiple references or point-instancer entries that all target that prototype:

     * Instanceable references: mark the referenced prim `instanceable = true` and reference it many times with different transforms / primvars.([NVIDIA Omniverse Docs][2])
     * `UsdGeomPointInstancer`: use one or more animated prototypes and scatter them on points (multi-instancer: many prototypes, many instances).([openusd.org][3])

   This gives you “N characters from 1 cache” with minimal additional data.

2. De-syncing / retiming instances
   To avoid all instances marching in lockstep, you can:

   * Apply `Sdf.LayerOffset` on references (offset/scale time) so each instance sees the same clip at a different phase or speed.([openusd.org][4])
   * Or, for more complex reuse/retime patterns (crowds, cycles), use value clips with different clip ranges / time mappings per prim.([openusd.org][1])

   Pipelines commonly use exactly this pattern for crowds and repeated FX.

3. Threads vs instances

   * USD’s design allows large scenes to be read and traversed efficiently and in parallel; multiple threads may read a stage concurrently, while concurrent writes to the *same* stage are disallowed.([openusd.org][5])
   * You do not normally create “one reader thread per character.” Instead:

     * You open a stage (or a small number of stages),
     * Let USD’s internals and your renderer/engine parallelize traversal, value resolution, and rendering.

   So from your perspective, “instancing many characters from one cache” is a composition/authoring pattern, not a threading pattern.

4. Mapping to Unreal Engine

   * If you import USD caches into Unreal as native assets (Geometry Cache, Skeletal Mesh + Animation), you then use Unreal’s own instancing (e.g., instanced meshes, duplicated actors, GPU instancing) to create many characters. The USD instancing is effectively baked into engine-native representations at import time.([Epic Developer Community Forums][6])
   * If you run a live USD stage in Unreal, the same USD instancing mechanisms apply; the underlying USD runtime handles how many threads it uses to read and compose the stage.

Summary:
Yes—USD is explicitly designed so that one animated prototype (your cached parametric animation) can be instanced many times with different transforms and time offsets to create many characters. Instancing is handled via USD composition (instanceable references or `UsdGeomPointInstancer`), while multi-threaded reading is handled internally by the USD and engine implementations rather than by manually spawning “reader threads per character.”
```js
[1]: https://openusd.org/dev/api/_usd__page__value_clips.html?utm_source=chatgpt.com "Sequencable, Re-timable Animated \"Value Clips\""
[2]: https://docs.omniverse.nvidia.com/usd/latest/learn-openusd/independent/modularity-guide/instancing.html?utm_source=chatgpt.com "Instancing — Omniverse USD"
[3]: https://openusd.org/dev/api/class_usd_geom_point_instancer.html?utm_source=chatgpt.com "UsdGeomPointInstancer Class Reference"
[4]: https://openusd.org/docs/567231471.html?utm_source=chatgpt.com "Transformations, Time-sampled Animation, and Layer Offsets"
[5]: https://openusd.org/dev/api/_usd__page__multi_threading.html?utm_source=chatgpt.com "Threading Model and Performance Considerations"
[6]: https://forums.unrealengine.com/t/unreal-4-24-universal-scene-description-usd-support/133680?utm_source=chatgpt.com "Unreal 4.24 Universal Scene Description (USD) support"
```
## OpenAI ChatGPT v1.2025.329 USD.For.Parametric.Animation.md

**Q: In Unreal Engine, I would like to use USD files or something more efficient to cache parametric animation data which can be instanced across the entire level to create the illusion of hundreds of interacting AI characters--which can be possessed without glitching mid sequence and driven by actual AI when the player is within range and then offloaded also without glitching once the player is out of range!**

**A:** Short answer:
Use USD only as an offline/cache format if you really need it; for runtime “hundreds of characters that can be possessed seamlessly”, Unreal’s MassEntity + regular animation assets + impostor/LOD strategy is the correct core, not runtime USD/VDB playback.

Below is a concrete architecture that matches your goals.

---

## 1. What USD is good/bad at for this use case

USD strengths in DCC / offline:

* Hierarchical scene description and instancing of geometry, materials, Xforms.
* Can store skeletal animation and parametric animation data as time-sampled attributes.
* Great for batching large scenes and reusing assets.

Limitations for your specific ask in UE runtime:

* Unreal’s USD plugin is primarily for import, live-link, and visualization — not a high-performance, runtime animation cache and instancer.
* Dynamic possession and switching to AI controllers mid-sequence is not a first-class concept in USD; you’d be writing a lot of glue to drive UE’s AnimInstances, Controllers, and replication off USD samples.
* You still pay the cost of skinning / evaluation in UE unless you go all the way to baked vertex animation or impostors.

So: you could write a custom system that reads USD time samples into param curves, but it is more complexity than benefit versus staying native.

---

## 2. High-level architecture in Unreal

You want three regimes around the player:

1. Near field:
   Full AI-driven characters (Controller + Behavior Tree/State Machine + AnimBP).

2. Mid field:
   Lightweight “scripted parametric” actors that:

   * Play pre-baked animation snippets (Montages, AnimSequences) or param-driven ControlRig.
   * Use a very cheap “fake AI state” (a few enums/parameters) but no heavy behavior tree.

3. Far field:

   * Niagara / impostor billboards / very low-poly skeletal meshes with simple looping anims, or even pure crowd sprites.

And you need seamless transitions between these states.

This maps well to:

* MassEntity / MassAI for large crowds.
* A small number of “real” AActors that get “bound” to Mass entities near the player.
* A stable state descriptor (“parametric state vector”) that exists independent of whether the entity is simulated by AI, by a pre-recorded sequence, or by a distant impostor.

---

## 3. Replace “USD param cache” with a parametric state model

Instead of caching raw transforms per frame, define a parametric state representation, e.g.:

```c++
// Example: "CrowdState" stored in Mass / data asset
struct FCrowdState
{
    uint8 BehaviorId;      // walk, talk, argue, idle, flee, etc.
    uint8 SubState;        // variations within a behavior
    float Phase;           // normalized [0,1) phase within that behavior’s cycle
    float Energy;          // speed, intensity modifier
    FVector2f LocalOffset; // small per-entity 2D offsets for variation
    uint32 Seed;           // RNG seed for procedural variation
};
```

Then build your animation system such that:

* An AnimInstance (or Control Rig) takes `FCrowdState` as input and:

  * Selects a montage / blendspace from `BehaviorId / SubState`.
  * Uses `Phase` to set the play position.
  * Uses `Energy` to scale play rate / additive layers.
* A “scripted crowd simulation” (no heavy AI) updates `BehaviorId`, `Phase`, `Energy`, etc., in Mass or a custom component.
* When you swap control (fake AI → real AI → fake AI), you only swap who writes to `FCrowdState`; the AnimInstance just keeps reading that data every tick.

You can store pre-baked “behavior graphs” in any format (JSON, curves, even USD if you really want), but what matters for performance and seamless switching is that, at runtime, you just update a small param struct, not skeletal data.

---

## 4. Instancing across the level

To create the “hundreds of interacting characters” illusion:

1. Represent each character as a Mass entity (or a lightweight UObject) holding:

   * `FCrowdState`
   * Position/orientation on a navigation graph or spline network.
   * High-level role flags (shopper, guard, bystander, etc.).

2. Drive interactions at a low frequency:

   * Update behavior graph at 5–10 Hz, not per frame.
   * Resolve simple interactions as param changes (e.g., “argue” = face each other, high `Energy`, specific animation set).

3. Rendering:

   * Bind only a subset of entities to actual skeletal meshes near the player.
   * Use “actor pooling”: a fixed pool of SkeletalMeshComponents is reused and bound to whatever entities are currently within a certain radius.
   * Mass Crowd / Mass Representation already does similar binding of Mass entities → visual actors.

This gives you the instancing you want: many logical crowd entities, a small pool of actual characters.

USD or other external cache can be used to author and preview parametric behavior graphs, but not as the primary runtime “per-frame data source.”

---

## 5. Seamless possession and AI handoff

Goal: Player approaches a scripted crowd member, presses “possess/interact,” and the character keeps moving naturally, now under real AI, without a visual pop.

Use this pattern:

1. Common runtime state:

   * `FCrowdState` lives in a component that both AI and the scripted system write to.
   * The AnimInstance only reads it.

2. Scripted mode (far/mid):

   * A “CrowdScriptController” (could be a Mass processor) drives `FCrowdState`:

     * Advances `Phase` according to a predetermined curve / param script.
     * Updates `BehaviorId` according to high-level rules or a preauthored timeline.

3. Handoff to AI:

   * When the player gets close:

     * Spawn or wake a real `AAIController` and attach to the pawn.
     * Initialize its blackboard/BT from the current `FCrowdState` (e.g., copy `BehaviorId` → “CurrentAction”).
     * Then switch the write authority:

       * `CrowdScriptController` marks this entity as “owned by AI” and stops writing to `FCrowdState`.
       * The AI controller starts writing AI-driven values to `FCrowdState` each tick.

   Critical detail:

   * Do not change the current animation sequence or phase on handoff.
   * AI uses the existing `Phase` and `BehaviorId` initially; only later transitions to new behaviors.

4. Handoff back to scripted:

   * When player leaves:

     * Freeze AI outputs to `FCrowdState`.
     * Capture snapshot of `FCrowdState`.
     * Switch entity back to `CrowdScriptController`, which resumes from that snapshot, optionally blending toward a more generic pattern over a few seconds.

Because the AnimInstance never sees a discontinuity in `FCrowdState` and remains the single consumer, you avoid “mid-sequence glitches”.

---

## 6. Where USD can still fit productively

If you still want USD in this workflow, good places are:

* Authoring:

  * Simulate complex multi-character scenes in DCC (Maya, Houdini) and export:

    * Character roles.
    * High-level behavior timelines as USD attributes.
* Bake step:

  * Write a custom build tool that:

    * Reads USD.
    * Converts behavior attributes into simplified param graphs / curves.
    * Outputs UE-native data assets (e.g., `UCurveFloat`, `UDataAsset` for `FCrowdBehaviorGraph`).

At runtime you then:

* Load only the compact UE data assets.
* Never touch USD in shipping builds.

USD is then a pipeline/interchange/cache format, not a runtime dependency.

---

## 7. Alternatives to USD/VDB for animation caching

Better-suited options baked into UE:

* Standard Skeletal Animation + Compression:

  * Highly optimized, supports instancing via Mass Crowd / pooling.
* Animation Blueprints + Pose Caching:

  * Cache frequently reused poses and blend between them.
* Control Rig:

  * Procedural parametric motion driven by a handful of parameters (your `FCrowdState`).
* Vertex Animation Textures (VAT):

  * For far-field crowds and cinematic loops: bake deformed vertex motion into textures.
* Niagara Impostors:

  * For very distant agents, where each “character” is just a sprite or simple mesh with a VAT animation.

This stack is generally more efficient and easier to ship than trying to stream per-frame transforms from USD or VDB.

---

## 8. Concrete UE blueprint for your goal

If you want a specific implementation path:

1. Adopt MassEntity + Mass Crowd:

   * Define a `FMassCrowdFragment` containing your `FCrowdState`.
   * Write Mass processors that update `FCrowdState` for mid/far entities.

2. Visual representation:

   * Use Mass Representation to bind entities to SkeletalMeshActors near the player.
   * Implement an AnimBP that takes `FCrowdState` (fed via an ActorComponent) and maps it to animation.

3. AI bridge:

   * For entities within a near radius:

     * Spawn a pawn + AIController.
     * Mirror the Mass entity’s `FCrowdState` into an Actor component that AI writes to.
     * When leaving radius, serialize `FCrowdState` back into the Mass entity and destroy or pool the pawn.

4. Authoring behaviors:

   * Start with simple hand-authored param curves / state machines.
   * If desired, later add a tool that imports USD-authored behavior timelines and compiles them into your param graphs.

---

If you want, next step I can sketch a minimal C++/Blueprint interface for `FCrowdState` and the handoff between Mass entities and pawn actors, including where to plug into the AnimInstance and AIController.

**Q: Does the AI Controller require spin up time? Does the nav mesh have to be pre established?**

**A:** Short:

1. An `AIController` itself does not need meaningful “spin-up time” – it can possess and start ticking the same frame, but your own initialization (Behavior Trees, Perception, EQS, etc.) can introduce hitches if you do too much at once.
2. For UE’s built-in pathfinding (`MoveTo`, `UAIBlueprintHelperLibrary::SimpleMoveToLocation`, etc.), a NavMesh must already be built/available in that area. If you move characters along splines or scripted paths, you can bypass NavMesh entirely for those agents.

---

### 1. Does an AIController require spin-up time?

Engine behavior:

* Spawn:

  * `AIController` is constructed and `BeginPlay()` runs once it enters the world.
  * Possession (`Controller->Possess(Pawn)`) can be immediate.
  * `Tick()` and movement components start working next frame as usual.

* Where “spin-up” comes from is usually:

  * Blackboard initialization (`UseBlackboard` / `InitBlackboard`).
  * Starting a `BehaviorTree` via `RunBehaviorTree`.
  * Registering `UAIPerceptionComponent` senses.
  * Any heavy setup in `BeginPlay` or `OnPossess`.

If these are simple and preconfigured:

* You can spawn a pawn + AIController and have it moving via `MoveTo` in the same or next frame without visible delay.

If you want to avoid hitches when “waking up” many at once:

* Pool controllers and pawns:

  * Create them on level load and keep them idle (no BT running).
  * On “promotion” from scripted state → AI:

    * Assign them an entity, copy state into blackboard, then enable BT.
* Stagger BT activation:

  * Activate a few BTs per frame.
  * Or use a simple “go to position” task first, then enable full behavior later.
* Avoid heavy one-shot work:

  * Move expensive asset loads and queries (e.g., EQS templates, many perception stimuli) out of `OnPossess` into global singletons or pre-warm phases.

So technically: no inherent multi-frame warmup is required; only your own logic and data loads can make it feel that way.

---

### 2. Does the NavMesh have to be pre-established?

For Unreal’s navigation system (`RecastNavMesh`):

* Yes, for built-in pathfinding:

  * `AIController` pathfollowing expects a valid nav data.
  * The NavMesh is generated inside `NavMeshBoundsVolume` regions.
  * In a typical setup, nav is baked at editor time and loaded with the level.
* Runtime generation:

  * `RecastNavMesh` can be configured for dynamic or “dynamic modifiers only”.
  * It can rebuild tiles at runtime for moving obstacles, level streaming, etc.
  * Full runtime generation over wide areas is expensive and can spike.

Recommendations for your scenario:

* Near-field AI (true AI controllers):

  * Pre-bake a NavMesh for the walkable regions around the player’s typical space (NavMeshBoundsVolume + “Static” or “DynamicModifiersOnly” runtime generation).
  * Ensure the NavMesh is present before you promote scripted entities to AI. If you use level streaming, promote only after the streamed-in tile’s nav data is ready.

* Mid/Far-field scripted crowd:

  * Does not strictly need NavMesh:

    * Drive them on splines, local graphs, or simple parametric motion.
    * You can still keep them logically “on nav” through your own 2D nav graph or precomputed routes.
  * This is ideal: use no NavMesh overhead for thousands of far-field agents.

* No NavMesh at all:

  * Only valid if:

    * You use custom pathfinding, or
    * Characters are on rails (spline paths, state machines that don’t need obstacle avoidance).
  * In that case, `AIController` can still exist but you would not call `MoveTo*`; you’d update character positions yourself.

---

### 3. Practical pattern for your crowd system

Tieing it back to your architecture:

* Far / Mid:

  * MassEntity or custom lightweight ECS:

    * Moves agents along scripted paths, local avoidance, etc.
    * No NavMesh required, no AIControllers.

* Near (promotion zone):

  * Make sure the NavMesh for this zone is pre-baked and loaded.
  * Either:

    * Pre-spawn a pool of pawn + AIController pairs, or
    * Spawn on demand but keep BT init and Perception minimal.
  * On promotion:

    * Attach a pooled controller to the visual pawn.
    * Copy the current parametric `CrowdState` into the AI’s blackboard.
    * Start the BT; let it continue from the current phase without changing the current animation state.

* Demotion:

  * Stop BT, copy out any state you care about.
  * Return the pawn/controller to pool or destroy.
  * Hand the logical entity back to scripted movement, continuing from the same position and `CrowdState`.

If you want, next step I can outline a concrete “promotion manager” in C++/Blueprint that checks nav readiness and handles allocating an AIController from a pool to avoid any runtime spikes.

**Q: Can a decent nav mesh encapsulating say 9-12 characters be configured as a real-time radius which tracks the player?**

**A:** Yes, you can have a “bubble” of NavMesh that follows the player and is large enough for ~9–12 characters, but you should do it with Navigation Invokers rather than literally moving a NavMesh volume every frame.

### Key idea

* Do **not** attach a `NavMeshBoundsVolume` to the player and move it around every tick. That forces constant tile rebuild and is expensive.
* Instead, use **Navigation Invokers** so the engine generates NavMesh tiles only within a configurable radius around the player (and/or other important actors). This gives you a moving NavMesh bubble with manageable cost.

### How to set it up

1. **Global nav settings:**

   * Place one or more large `NavMeshBoundsVolume`s that cover the whole walkable world area (or at least all potential player paths).
   * In Project Settings → Navigation System:

     * Set `Runtime Generation` on your `RecastNavMesh` to `Dynamic` or `DynamicModifiersOnly`.
     * Enable **“Generate Navigation Only Around Navigation Invokers”**.

2. **Attach an invoker to the player:**

   * Add a `UNavigationInvokerComponent` (or Blueprint equivalent) to the player pawn/character.
   * Configure:

     * `Tile Generation Radius`: radius around the player where NavMesh tiles should exist.
     * `Tile Removal Radius`: radius at which old tiles are discarded behind the player.

   Example ballpark for your “9–12 characters” bubble:

   * Tile size ~300–600 Unreal units.
   * `Tile Generation Radius` ~2000–3000 units (20–30 m).
   * That gives enough surface for a small group of agents to move, flank, and avoid each other.

3. **Optionally add more invokers:**

   * If some AI can be far from the player but still need pathfinding, give them their own `NavigationInvokerComponent`.
   * The engine will then maintain separate nav bubbles around them as well.

### Behavior and performance characteristics

* The NavMesh is still world-aligned and tile-based; the **generated region effectively tracks your invoker**, but tiles themselves don’t move — they’re created and destroyed as the player moves.
* Rebuilds happen per tile and are asynchronous; with a reasonable tile size and radius, this is smooth for a bubble that supports ~9–12 active AI.
* `AIController::MoveTo` / path following just works as long as:

  * The agent’s current position and destination are inside the generated NavMesh region.
* For your crowd architecture:

  * Run **full AI controllers only inside the nav bubble**.
  * Outside that bubble, keep agents on splines / scripted paths without using the NavMesh at all.

So, yes: a “real-time radius” of usable NavMesh around the player is a supported pattern in Unreal, implemented via Navigation Invokers rather than a physically moving NavMesh volume.
