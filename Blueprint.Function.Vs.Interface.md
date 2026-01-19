## OpenAI ChatGPT v1.2026.006 Blueprint.Function.Vs.Interface.md

**Q: How does an unreal engine blueprint function differ from a blueprint interface?**

**A:** **Result:** A Blueprint Function implements behavior; a Blueprint Interface defines a behavior contract without implementation.

**Differences**

**Blueprint Function**

* Concrete logic with execution flow.
* Lives in a Blueprint class (Actor, UObject, Blueprint Function Library).
* Can read/write state, call other functions, spawn objects, and access engine context.
* Callable only on compatible classes or via static calls (Function Library).
* Supports latent actions (Delay, Timelines) when allowed by context.

**Blueprint Interface**

* Declaration only: function signatures, no logic.
* No variables, no state, no graphs.
* Any Blueprint class can implement it, regardless of inheritance.
* Enables polymorphism and loose coupling.
* Calls are safe even when the concrete class is unknown at compile time.

**When to use**

* Use a **Blueprint Function** for reusable or class-specific behavior.
* Use a **Blueprint Interface** to guarantee that disparate classes respond to the same message without shared ancestry.

**Rule of thumb**

* *Need shared code?* Function.
* *Need shared capability across unrelated classes?* Interface.

Expanded details, edge cases (performance, casting costs, latent support), or Unreal Engine Blueprint architectural patterns available on request.

**Q: So a blueprint function should not be subclassed?**

**A:** **Result:** Correct. A Blueprint *function* itself is not subclassed; the *owning Blueprint class* is.

**Clarification**

* Functions are members, not types.
* Inheritance applies to Blueprint **classes** (Actor, UObject, etc.), not to individual functions.

**Implications**

* A child Blueprint inherits all non-private functions of its parent class.
* You can **override** a function in a child Blueprint (if marked overridable), but you are not subclassing the function—only replacing its implementation in the subclassed class.
* Blueprint Function Libraries are static and **cannot** be overridden or specialized via inheritance.

**Contrast**

* Blueprint Interface functions are also not subclassed.
* They are *implemented* independently by each class that adopts the interface.

**Mental model**

* Class inheritance → function inheritance.
* Function override ≠ function subclass.
* Interface → capability contract, not behavior inheritance.

If you want, I can map this directly to Unreal Engine Blueprint patterns (UObject utility base class vs interface vs function library) with concrete do/don’t examples.
