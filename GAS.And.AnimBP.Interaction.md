## OpenAI ChatGPT v1.2026.006 GAS.And.AnimBP.Interaction.md

**Q: Is the unreal engine gameplay ability system migrating away from animation blueprints for decision logic or the GPU?**

**A:** No. Gameplay Ability System (GAS) is not “migrating away” from Animation Blueprints (AnimBPs), and it is not moving decision logic onto the Graphics Processing Unit (GPU).

* GAS scope: GAS is a gameplay framework (abilities, effects, attributes). Abilities commonly *trigger* animation (typically via montages / ability tasks), but GAS does not replace the animation graph or the AnimBP’s pose-generation role. Epic’s GAS overview explicitly frames abilities as executing ability events including “playing animations.” ([Epic Games Developers][1])
* AnimBP scope: AnimBPs remain the primary runtime system for evaluating animation graphs, blending, and producing final poses; current engine releases continue to add/maintain AnimBP features (notably in recent release notes). ([Epic Games Developers][2])
* “Decision logic” trend: The direction is better separation of concerns, not deprecating AnimBPs. High-level gameplay decisions (intent, ability selection, cooldown gating, targeting) generally belong in Character/Controller/GAS; AnimBPs should mostly consume already-decided state (speed, gait, aim offsets, tags, montage state) and produce poses. This is consistent with how GAS is documented (abilities own gameplay execution, including when to play animation), while AnimBPs own pose evaluation. ([Epic Games Developers][1])
* GPU: Animation *skinning* is commonly done on the GPU, but the “decision logic” (branching, state machine transitions, ability activation rules) remains CPU-side. Unreal Engine’s ongoing animation work emphasizes optimization and workflow, not pushing gameplay/decision code to GPU. ([Epic Games Developers][3])

Practical takeaway:

* Put “what should the character do” in GAS/gameplay code (often keyed off Gameplay Tags (GT) and ability state).
* Put “how do we pose/blend to represent that” in AnimBP (state machine/blend spaces/linked layers), with minimal branching beyond pose concerns.
* Let GAS drive montages (and tag state), and let AnimBP react to montage + tag/variables rather than deciding abilities.
```
[1]: https://dev.epicgames.com/documentation/en-us/unreal-engine/understanding-the-unreal-engine-gameplay-ability-system?utm_source=chatgpt.com "Understanding the Unreal Engine Gameplay Ability System"
[2]: https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-5-7-release-notes?utm_source=chatgpt.com "Unreal Engine 5.7 Release Notes"
[3]: https://dev.epicgames.com/documentation/en-us/unreal-engine/animation-optimization-in-unreal-engine?utm_source=chatgpt.com "Animation Optimization in Unreal Engine"
```
**Q: Is the Movers logic run on the gpu or does its performance hinder gpu bound workloads in any way?**

**A:** Mover runs on the Central Processing Unit (CPU), not the Graphics Processing Unit (GPU).

* Mover simulation hooks are “guaranteed to be on the game thread,” i.e., CPU-side. ([Epic Games Developers][1])
* In the physics-driven examples, Mover can run with asynchronous physics ticking and prediction; that is still CPU (physics thread / game thread), with the game thread interpolating toward the latest physics state. ([Epic Games Developers][2])

Impact on GPU-bound workloads:

* It does not directly consume GPU time, so it does not “hinder” GPU-bound rendering in the direct sense.
* It can still reduce overall frame rate if Mover pushes the frame to become CPU-bound (e.g., game thread or physics thread becomes the limiting factor). Unreal’s profiling model is that frame rate is limited by the slowest stage (CPU or GPU), so adding CPU cost matters only if it becomes the bottleneck. ([Epic Games Developers][3])

Practical diagnostic:

* If `stat unit` / Unreal Insights shows GPU time > game thread / render thread time, Mover cost typically will not change frame rate until it shifts the bottleneck to CPU.
* If you see game thread or physics thread time rising above GPU time, Mover can become the limiter.
```
[1]: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Plugins/Mover?utm_source=chatgpt.com "Mover | Unreal Engine 5.7 Documentation"
[2]: https://dev.epicgames.com/documentation/en-us/unreal-engine/mover-examples-in-unreal-engine?utm_source=chatgpt.com "Mover Examples in Unreal Engine"
[3]: https://dev.epicgames.com/documentation/en-us/unreal-engine/introduction-to-performance-profiling-and-configuration-in-unreal-engine?utm_source=chatgpt.com "Introduction to Performance Profiling and Configuration"
```
**Q: Compare Mover to the gameplay ability system!**

**A:** Mover and Gameplay Ability System (GAS) solve different problems and sit in different layers of the stack.

Mover vs GAS

1. Primary responsibility

* Mover: character/pawn locomotion simulation (movement modes, collision/contacts, optionally physics-driven movement, prediction backends). ([Epic Games Developers][1])
* GAS: gameplay “verbs” and state (abilities, cooldowns/costs, effects, attributes, tag-driven gating/cancellation, coordination of animation/visual/audio). ([Epic Games Developers][2])

2. What they output

* Mover: a new transform/velocity (and related movement state) each tick.
* GAS: authoritative gameplay state transitions (ability activated/ended), attribute deltas via Gameplay Effects, and events; often triggers animation (usually montages) and waits on gameplay/animation events. ([Epic Games Developers][3])

3. Threading/execution model

* Mover: CPU-side simulation; examples describe physics sim running ahead of the game thread with the game thread interpolating representation. ([Epic Games Developers][4])
* GAS: CPU-side gameplay framework; executes abilities/effects on server and/or predicting client depending on configuration, but fundamentally game-code execution.

4. Networking and prediction

* Mover: explicitly concerned with movement prediction/replication; documentation/examples discuss physics prediction and interpolation behavior for network play. ([Epic Games Developers][4])
* GAS: explicitly concerned with gameplay prediction/replication (client-predicted abilities, server authority, rollback-style reconciliation patterns); core docs frame it as coordinating mechanics/data-driven elements across networked games. ([Epic Games Developers][2])

5. Data model

* Mover: movement modes, movement state, contact resolution; configured via movement-mode classes (see API surface). ([Epic Games Developers][1])
* GAS: Abilities + Gameplay Effects + Attributes (Attribute Sets) + Gameplay Tags (GT) as the core vocabulary. ([Epic Games Developers][2])

6. Typical integration pattern

* GAS drives “intent” (sprint, dodge, mantle, knockback, stun), sets/clears tags and applies effects; triggers a montage and listens for gameplay/animation events. ([Epic Games Developers][5])
* Mover consumes that intent as inputs to movement (e.g., select movement mode, change max speed/accel, apply root-motion-like impulses, disable input during stun), then simulates motion.

Rule of thumb

* Use Mover when the hard part is locomotion quality/robustness/predicted movement.
* Use GAS when the hard part is gameplay orchestration (costs/cooldowns, stacking effects, attribute math, tag-gated interactions, replicated ability execution).

They are complementary: GAS decides “what you are allowed/trying to do,” Mover computes “how you physically move when doing it.”
```
[1]: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Plugins/Mover?utm_source=chatgpt.com "Mover | Unreal Engine 5.7 Documentation"
[2]: https://dev.epicgames.com/documentation/en-us/unreal-engine/understanding-the-unreal-engine-gameplay-ability-system?utm_source=chatgpt.com "Understanding the Unreal Engine Gameplay Ability System"
[3]: https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-effects-for-the-gameplay-ability-system-in-unreal-engine?utm_source=chatgpt.com "Gameplay Effects"
[4]: https://dev.epicgames.com/documentation/en-us/unreal-engine/mover-examples-in-unreal-engine?utm_source=chatgpt.com "Mover Examples in Unreal Engine"
[5]: https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-ability-tasks-in-unreal-engine?utm_source=chatgpt.com "Gameplay Ability Tasks in Unreal Engine"
```
**Q: Explain Choosers, its effect on cpu/gpu, how it compares to Movers, GAS, and the problem it solves!**

**A:** Choosers (Chooser Plugin / Dynamic Asset Selection) is a data-driven runtime selection system: given a “context” (objects/struct data), it evaluates a Chooser Table and returns the best-matching asset (commonly animations, but not limited to animations). ([Epic Games Developers][1])

What problem Choosers solves

* Replaces large, brittle branching graphs (Animation Blueprint (AnimBP) state machine branches, Blueprint (BP) switch chains, nested ifs) with a declarative table of rules + weights/priorities.
* Scales better when “the same action” has many variants (hit reacts by body part, locomotion variants by gait/weapon stance/surface, additive overlays by gameplay state), because you extend rows/columns instead of rewriting control flow. Epic’s documentation frames it as selecting among “various iterations” based on context. ([Epic Games Developers][1])

CPU/GPU impact

* Choosers evaluation is CPU work (rule evaluation, comparisons, table traversal).
* It does not execute on the GPU and does not directly increase GPU frame time.
* Practical impact is only via CPU budget: if you evaluate choosers frequently (per-tick per-character, multiple tables per character), it can push you toward a game-thread / worker-thread CPU bottleneck, which then caps frame rate even if the GPU has headroom (standard Unreal profiling model). ([Unreal Engine][2])

How it compares

1. Choosers vs Mover

* Mover: locomotion/movement simulation (transform/velocity, collision, prediction) on CPU; it is the “physics/kinematics of moving.” (This is movement-domain computation.)
* Choosers: selection/orchestration of assets/behaviors based on context; it is “pick which variant to use.” (This is decision/selection-domain computation.)
* Relationship: Mover produces state (speed, acceleration, movement mode) that commonly becomes chooser inputs; choosers pick the locomotion animation set (or Pose Search Database (PSD) / motion-matching configuration) matching that state. The public roadmap explicitly calls out Motion Matching Chooser integration via a PoseSearch field in Choosers (experimental). ([portal.productboard.com][3])

2. Choosers vs Gameplay Ability System (GAS)

* GAS: authoritative gameplay verbs and state transitions (abilities, cooldowns/costs, Gameplay Effects, attributes, Gameplay Tags (GT)) and their replication/prediction model. ([Epic Developer Community Forums][4])
* Choosers: a generalized selector; it does not own authority, costs/cooldowns, stacking effects, or replication semantics.
* Relationship: GAS/GT often provide the “context” (weapon equipped tag, stance tag, damage type, crowd-control state). Choosers then select the correct animation montage, hit react, VFX, or data asset for the current tagged state. ([Epic Games Developers][1])

3. Choosers vs AnimBP decision logic

* AnimBP: produces poses; modern Unreal supports parallel animation update/evaluation on worker threads for performance. ([Epic Games Developers][5])
* Choosers: can be invoked from BP/AnimBP to select assets, reducing branching complexity inside AnimBP graphs.
* Pattern: keep high-level “what variant” selection in Choosers (table), keep “pose synthesis/blending” in AnimBP (graph), and keep “gameplay authority” in GAS/gameplay code.

When Choosers is the right tool

* Many-variant selection problems where inputs are multidimensional and change over time (tags + locomotion state + environment + weapon + damage metadata).
* You want designers to iterate by editing tables, not graphs/code.

When it is not sufficient

* You need network authority/prediction rules (use GAS / gameplay code).
* You need to actually simulate motion/collision/predicted movement (use Mover / movement components).
* You need pose generation/blending/IK (Inverse Kinematics) (use AnimBP/animation systems; Choosers can only pick inputs).

Performance guidance (practical)

* Prefer event-driven evaluation (on state changes) over unconditional per-tick evaluation where possible.
* If used per tick, keep context extraction cheap and minimize the number of tables evaluated per character per frame; verify with Unreal Insights / stat profiling (game thread and animation worker threads). ([Unreal Engine][2])
```
[1]: https://dev.epicgames.com/documentation/en-us/unreal-engine/dynamic-asset-selection-in-unreal-engine?utm_source=chatgpt.com "Dynamic Asset Selection in Unreal Engine"
[2]: https://www.unrealengine.com/es-ES/blog/how-to-improve-game-thread-cpu-performance?utm_source=chatgpt.com "How to improve game thread CPU performance in ..."
[3]: https://portal.productboard.com/epicgames/1-unreal-engine-public-roadmap/c/2157-motion-matching-chooser-integration-experimental-?utm_source=chatgpt.com "Motion Matching Chooser Integration (Experimental)"
[4]: https://forums.unrealengine.com/t/game-animation-sample-migration/2205244?utm_source=chatgpt.com "Game animation sample migration - Programming & Scripting"
[5]: https://dev.epicgames.com/community/learning/knowledge-base/xBZp/unreal-engine-performance-tips-tricks-animation?utm_source=chatgpt.com "Performance Tips & Tricks - Animation"
```
