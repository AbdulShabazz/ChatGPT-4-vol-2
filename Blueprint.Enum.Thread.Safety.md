## OpenAI ChatGPT v1.2025.350 Blueprint.Enum.Thread.Safety.md

**Q: When should I make an unreal engine blueprints enum class threadsafe?**

**A:** You generally **do not (and cannot meaningfully) make a Blueprint enum itself “thread-safe.”** Thread-safety in Unreal Engine usually applies to **the code that reads/writes the enum value**, not the enum type.

When thread-safety matters is when the enum value is accessed from **non–Game Thread** contexts (common examples below). In those cases, the rule is: **reading an enum value is fine; sharing and mutating it across threads is what needs explicit synchronization.**

When you need to treat enum usage as thread-sensitive

1. Animation Blueprint worker threads

* Anim Graph evaluation and parts of animation update can run on worker threads.
* Constraint: any logic executed there must be “thread-safe” (no touching `UObject` state that is not thread-safe, no world queries, no component mutation, etc.).
* Practical implication: if you have Blueprint-callable functions that *return/compute* an enum used in Anim Graph evaluation, the *function* must be authored to be thread-safe (typically via C++ patterns; Blueprint graphs themselves are not a strong guarantee).

2. Async tasks / Task Graph / `AsyncTask`, `UE::Tasks`, `FAsyncTask`

* If an enum value is shared between an async task and the Game Thread, you need synchronization for writes/hand-off.

3. Mass/Gameplay systems that run parallel work

* Same issue: shared mutable state needs synchronization.

What to do instead

A) If the enum is only read on worker threads (no concurrent writes)

* Do nothing special.
* Ensure the value is produced on the Game Thread and then handed off immutably (copy into thread-local data / proxy struct).

B) If the enum is written on one thread and read on another (shared state)
Use one of these patterns:

* **Atomic storage**: store as an integral atomic and cast.

  * Example approach: `TAtomic<uint8>` (or `std::atomic<uint8_t>`) holding the enum’s underlying value; cast to/from the enum.
  * This is appropriate for simple state flags.

* **Locking**: protect the shared variable with a mutex/critical section.

  * Appropriate if the enum change is part of a larger shared state update.

* **Message passing / handoff**: push updates through a thread-safe queue to the Game Thread, then apply.

  * Best when the enum change is part of gameplay state that should ultimately live on the Game Thread.

C) If you think “BlueprintThreadSafe” is what you mean

* That metadata applies to **functions** (primarily in C++ Blueprint function libraries / anim nodes), not enums.
* Mark the **function** that reads/derives the enum as thread-safe only if it:

  * does not touch non-thread-safe `UObject` state,
  * does not access the world, components, or mutable globals,
  * operates purely on inputs / thread-local or immutable data.

Rule of thumb

* If the enum lives on a `UObject` (Actor/Component/AnimInstance) and you might touch it off the Game Thread: **don’t**; instead copy it into a thread-safe proxy or synchronize access.
* If it’s a local value copied into the worker-thread context: it’s already safe.

If you describe where the enum is used (Anim Graph evaluation vs gameplay vs async task) and how it is stored (UObject variable vs proxy struct), I can recommend the exact pattern (atomic vs proxy vs queue).

**Q: I mutate the enum through a custom event while another function reads and mutates the enum value via ontick!**

**A:** Treat it as **not thread-safe** unless you can prove both call sites are on the **Game Thread**. If both are on the Game Thread, no thread-safety work is needed; you have an **order-of-execution / re-entrancy** problem, not a threading problem.

What is likely happening

* A Custom Event mutates the enum at some point during the frame.
* Tick reads it and may also mutate it.
* Result: race-like behavior from nondeterministic ordering (event vs tick), repeated writes, or state transitions being applied twice.

Correct patterns (Blueprint-safe)

1. Single-writer, multi-source inputs (recommended)

* Make Tick the **only place that writes** the enum.
* Custom Events only write “intent” variables (booleans/requests) or enqueue a pending request.
* Tick consumes intent(s) and performs a single state transition.

Blueprint sketch:

* Variables:

  * `State` (your enum)
  * `PendingState` (same enum) or `bStateChangeRequested` + `RequestedState`
* Custom Event:

  * Set `PendingState = NewState` (or set request vars)
* Tick:

  * If `PendingState != None/Invalid` and `PendingState != State`:

    * `State = PendingState`
    * Clear `PendingState`
  * Then run state logic based on `State`

This guarantees deterministic “one transition per frame” semantics.

2. Edge-triggered transition gate (if you must write from both)

* Add a guard that prevents Tick from mutating the enum during the same frame the event changed it.
* Variables:

  * `LastStateWriteFrame` (int)
* On any write:

  * `LastStateWriteFrame = Get Frame Count`
* Tick:

  * If `LastStateWriteFrame == Get Frame Count`: do not write (read-only this frame)

This prevents ping-pong and double-applies inside one frame.

3. Use a state machine function with explicit transitions

* Centralize all mutations into `SetState(NewState)` that:

  * Rejects no-op transitions (`if NewState == State return`)
  * Optionally validates transition table (`Allowed[State][NewState]`)
  * Calls `OnExitState(State)` then sets then `OnEnterState(NewState)`
* Tick and events both call `SetState`, but the mutation logic is centralized and guarded.

Thread-safety decision rule for your case

* If both the Custom Event and Tick are standard Blueprint on an Actor/Component: they run on the **Game Thread**. No “threadsafe enum” concept applies.
* If the reader/mutator is inside **Animation Blueprint evaluate** paths or an async task: then you must not read/mutate shared `UObject` state there; copy to a proxy or synchronize via message passing.

Fast way to tell if you have real multithreading

* If the mutating Tick is `Event Tick` on an Actor/Component: Game Thread.
* If the read/mutate happens in an Anim Graph node evaluation / “thread-safe” anim function: may be worker thread; do not touch shared mutable state there.

Minimum fix

* Implement pattern (1): Custom Event sets `PendingState`, Tick is sole writer applying `State = PendingState`.

If you tell me where the Tick function lives (Actor Tick vs AnimInstance Update vs Anim Graph evaluation) I can classify it as “execution ordering” vs “actual cross-thread hazard” and give the exact Blueprint graph shape.

**Q: Then is there a possible race between ontick and the custom event setting and resetting the intent bool?**

**A:** No, not a thread race if both execute on the Game Thread. Yes, you can still get a frame-level ordering hazard (lost update) if you use a single “intent bool” that is set and cleared in different places.

Common failure modes (single thread, same frame)

* Lost intent: Custom Event sets `bIntent=true`, Tick reads it and clears it, then later in the same frame another Custom Event sets it again (or vice versa). Outcome depends on call order.
* Double-consume: Tick runs multiple times (rare) or multiple code paths clear the same intent, causing you to miss transitions.
* Re-entrancy: Event triggered during Tick processing changes the intent/state mid-Tick, producing inconsistent state.

Deterministic fixes

1. Latch + consume in one place (recommended)

* Only Tick clears the intent.
* Events only set it (never clear).
* Tick copies then clears once per frame.

Blueprint logic in Tick:

* `LocalIntent = bIntent`
* `bIntent = false`
* If `LocalIntent`: perform transition

This prevents “Tick clears while Event clears” conflicts. It does not prevent “Event sets after Tick already consumed this frame,” but that’s usually acceptable (it triggers next frame). If you need “same frame immediate,” use (3).

2. Use an incrementing token instead of a bool (robust)

* `IntentCounter` (int)
* Event: `IntentCounter++`
* Tick: if `IntentCounter != LastConsumedCounter`: consume; set `LastConsumedCounter = IntentCounter`
  No lost updates regardless of how many events fire between ticks.

3. Queue requests (best when multiple distinct intents)

* `PendingStates` array or a simple ring buffer (Blueprint-friendly: array + push).
* Event: `PendingStates.Add(NewState)`
* Tick: while `PendingStates.Num>0`: pop(0) or pop last and apply transitions (optionally collapse to last request only).

4. Frame stamp gate (simple)

* `IntentFrame` (int), `ConsumedFrame` (int)
* Event: `IntentFrame = Get Frame Count`
* Tick: if `IntentFrame > ConsumedFrame`: consume; `ConsumedFrame = IntentFrame`
  Works like a bool but can’t be “re-cleared” incorrectly within the same frame.

Practical recommendation

* If the intent is “do X at least once soon,” use (1).
* If multiple events can happen between ticks and you must not drop any, use (2) or (3).
* If you only care about the latest requested state, use a single `PendingState` enum (overwrite) and apply once per tick.

Threaded exception
If either side runs off the Game Thread (async task or Anim Graph evaluation), then yes you can have a real data race; Blueprint bool/enum writes are not atomic or synchronized. In that case, route intent to the Game Thread (queue + `Async Task (Game Thread)`), or store in atomic/in C++ and read via thread-safe proxy.

If you want, describe where the Custom Event is fired from (input, overlap, anim notify, async completion) and where the Tick runs (Actor/Component vs AnimInstance) and I’ll pick the minimal correct pattern.
