## OpenAI ChatGPT v1.2025.336 Event.Tick.Performance.In.UE.md

**Q: In Unreal Engine Blueprints, is it more performant to create one global Event Tick node which fires an event (returning the current game time in seconds!) which any object can subscribe to?**

**A:** No. A single “global tick + broadcast to subscribers” is not inherently more performant than letting actors tick, and it can be worse.

Key points

* Cost is dominated by “how many things execute per frame,” not where the tick originates. If 500 objects subscribe and run logic every frame, you still pay ~500 calls per frame (plus delegate/broadcast overhead).
* A multicast delegate/event dispatch from a manager adds overhead (broadcast + invocation + Blueprint virtual dispatch) on top of whatever each subscriber does.
* You only win if the pattern lets most objects keep `Tick` disabled (so they never enter the engine’s tick pipeline) and only a small subset subscribes/updates at any time.

Better performance patterns (Blueprint-first)

* Prefer event-driven updates (overlaps, input, animation notifies, perception, gameplay events) instead of per-frame polling.
* Use `Set Actor Tick Enabled(false)` by default; enable tick only while needed (in-range, active state, visible, etc.).
* Use `Set Actor Tick Interval` (e.g., 0.05–0.2 s) for things that do not need per-frame updates.
* Use `Set Timer by Event/Function` for periodic work; timers scale better than per-frame updates for many actors.
* For global “time in seconds,” call `Get Game Time in Seconds` (or `Get Time Seconds`) where needed; caching and rebroadcasting time rarely saves anything.

When a global tick manager makes sense

* You need a fixed-step “simulation clock” (e.g., 20 Hz) and can update *only* registered active objects on that schedule (not per frame).
* You want centralized throttling (distance-based buckets, staggered updates, LOD-style update rates).

Rule of thumb

* If “most objects would subscribe” and run every frame: do not centralize; it will not be faster.
* If “few objects need updates at any given time” and you can keep tick disabled elsewhere: a manager (often timer-driven, not per-frame) can be a net win.
