## OpenAI ChatGPT v1.2025.336 Passing.Timeline.Reference.md

**Q: Although its not possible to host a timeline in a component can I pass a reference of it to the component?**

**A:** Yes. You can pass a reference to the owning Actor’s timeline (or more commonly, pass the *data it produces*) into a component; you cannot “re-parent” the timeline into the component, and you can’t hold a typed “Timeline” object reference in Blueprints the way you can for Actors/Components.

Practical patterns that work in Blueprints:

1. Preferred: component exposes an API; owner drives it

* Actor owns the Timeline.
* Timeline Update pin calls a component function like `OnTimelineUpdate(Alpha)` or `SetBlend(Alpha)`.
* Timeline Finished pin calls `OnTimelineFinished()` if needed.
  This avoids any need for a timeline reference.

2. If you need the component to “control” playback: pass an interface, not the timeline

* Define a Blueprint Interface (e.g., `BPI_TimelineControl`) with functions like `Play`, `Reverse`, `Stop`, `SetPlayRate`.
* Actor implements it and internally calls its timeline nodes.
* Component stores an interface reference (to the owner) and invokes those functions.

3. If you need curve/time evaluation inside the component: pass the assets

* Pass `CurveFloat/CurveVector` references (assets) into the component.
* Component evaluates curves using `Get Float Value` (etc.) given a time value you provide (from owner tick/timer), or owner provides time.

Key constraint
Blueprints do not expose a stable object you can store as “Timeline reference” and later invoke `Play/Stop` on from another Blueprint. Timeline nodes are graph-bound to the Blueprint that defines them.

If you describe what the component needs to do (consume alpha vs start/stop/seek), I can give the smallest clean graph pattern.
