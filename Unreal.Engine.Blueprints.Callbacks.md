## OpenAI ChatGPT v1.2026.006 Unreal.Engine.Blueprints.Callbacks.md

**Q: Does unreal engine blueprints support passing functions as callbacks?**

**A:** Yes — but **not as raw function pointers**. Blueprints pass “callbacks” using **delegates**, not functions themselves.

Here is how callbacks are expressed in Blueprint:

**1) Event Dispatchers (standard pattern) — recommended**

* Create a **Dynamic Multicast Delegate** in a Blueprint.
* Bind to it with **Bind Event to [Object]** or **Add**.
* Broadcast it with **Call**.
* This is the closest analogue to a callback.

**2) Interfaces (indirect callbacks)**

* Define a Blueprint Interface.
* Call the interface function on a target actor/widget.
* The receiver implements the response logic.
* Useful when the callee must control *who* handles the callback.

**3) Timers by function name (limited form)**

* `Set Timer by Function Name`.
* Works like a deferred callback but only to a function on the same object.

**4) Latent nodes with completion pins**

* Examples: Delay, Timeline, Async Load, MoveTo, etc.
* Not “passed functions,” but still a callback pattern.

**What you cannot do**

* You cannot pass an arbitrary function as a value like `Func f = MyFunc`.
* Blueprints lack first-class function references.

**Short rule**

* If you want callbacks: use **Event Dispatchers**.
* If you want polymorphic responses: use **Interfaces**.
