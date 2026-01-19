## OpenAI ChatGPT v1.2026.006 Duplicating.Blueprint.Explanation.md

**Q: Does duplicating an unreal engine blueprint make it a blueprint instance?**

**A:** **Answer:** No. Duplicating an Unreal Engine Blueprint asset creates a **new Blueprint class**, not a Blueprint instance.

**Explanation (minimal):**

* **Duplicate Blueprint asset (in Content Browser):**
  Creates a separate Blueprint class with its own Class Default Object (CDO). Changes do not propagate between the original and the duplicate.
* **Instance:**
  Created by placing a Blueprint class into a level (or spawning it at runtime). Instances share the class but have per-instance overridden properties.
* **Child Blueprint (Create Child Blueprint Class):**
  Creates an inherited class. Parent changes propagate unless overridden.

**Summary table:**

| Action                     | Result                  |
| -------------------------- | ----------------------- |
| Duplicate Blueprint asset  | New class (independent) |
| Place Blueprint in level   | Instance                |
| Spawn Blueprint at runtime | Instance                |
| Create Child Blueprint     | Inherited class         |

If you want shared behavior with divergence, use **child Blueprints**. If you want per-object variation at runtime, use **instances**.

**Q: Ahh, okay, so create an instanceable child blueprint.**

**A:** Yes: create a **child Blueprint class**, then create **instances** of that child.

Minimal workflow:

1. Content Browser → right-click parent Blueprint → **Create Child Blueprint Class**.
2. Place the **child** into the level (or spawn it) to make instances.
3. Put shared logic/defaults in the **parent**; specialize in the **child** (defaults, components, overridden events/functions).
4. For per-instance tweaks, change properties on the placed actor (Details panel) or set them after spawn.

Key distinctions:

* Child Blueprint = new class that inherits from parent.
* Placed/spawned actor = instance of that class.

**Q: What happens if I delete the parent while a child blueprint exists? What if I delete the instanceable blueprint while an instance persists?**

**A:** **Result-first summary:**

* Deleting a **parent Blueprint** while children exist **breaks inheritance**; children become invalid and must be reparented or will fail to compile/load.
* Deleting a **Blueprint class** while **instances exist** leaves **orphaned instances** that fail to load and are typically removed or replaced with placeholders on map load.

**Details:**

### Delete parent Blueprint while child exists

* Children reference the parent’s generated class.
* On deletion:

  * Editor warns about dependencies.
  * If forced:

    * Child Blueprints lose their parent reference.
    * Compile errors occur; graphs/components inherited from parent are lost.
    * You must **Reparent Blueprint** to a valid class to recover.
* Best practice: reparent children **before** deleting the parent.

### Delete instanceable Blueprint (class) while instances persist

* Level instances reference the Blueprint class asset.
* On deletion:

  * Editor warns about level references.
  * If forced:

    * Instances cannot resolve their class on load.
    * They are usually **removed**, or replaced by an **“Unknown/REINST” placeholder**.
    * Any per-instance data is lost.
* There is no supported way to keep instances without the class.

**Safe workflows:**

* Use **Reparent Blueprint** for children.
* Use **Replace References** (Content Browser) to swap a class before deletion.
* Migrate instances by selecting them in-level and changing their class **before** deleting the original Blueprint.

**Rule of thumb:**
Classes own instances; parents own inheritance. Deleting either invalidates dependents unless explicitly migrated first.
