## OpenAI ChatGPT v1.2026.006 Scalable.Blueprint.Messaging.md

**Q: Rather than flooding a blueprint interface with messages to every subscriber to which it only an individual must respond to, I decided to implement a unique blueprint interface between the third person character blueprint and each subscriber. The solution unfortunately is not scalable!**

**A:** Per-subscriber Blueprint Interface (BPI) instances are the wrong abstraction. Use one shared contract (one BPI) plus addressing/routing (who the message is for) so only the intended receiver handles it, without N broadcast spam and without N unique interfaces.

What to do instead (scalable patterns)

1. Single BPI + targeted calls (no broadcast)

* Store a reference to the intended receiver (Actor/Component) and call the interface on that one object.
* Keep a registry on the ThirdPersonCharacter (or a manager Subsystem) of “subscribers” as interface-implementers.
* Use a key to resolve “who”:

  * Gameplay Tag (recommended) or Name or GUID.
* Flow:

  * Subscriber registers: RegisterSubscriber(Tag, ObjectRef)
  * Character looks up: GetSubscriber(Tag) -> ObjectRef
  * Character calls: BPI_Notify(ObjectRef, Payload)

Result: O(1) dispatch, single interface, no spam.

2. Event Dispatcher per event type + filtered payload
   If you still want multicast for some events:

* Broadcast once, but include an Address/Channel in the payload (Gameplay Tag / ID).
* Each subscriber early-outs unless Address matches.
* To avoid per-tick overhead:

  * Only broadcast on state changes.
  * Pre-bind only relevant subscribers per channel (map Tag -> dispatcher) if needed.

Result: still “broadcast,” but cheap and logically filtered; scales with event frequency not subscriber count.

3. Use Actor Components as subscribers
   Instead of “many unrelated actors listening”:

* Create a UActorComponent (Blueprintable) like BP_CombatSubscriber, BP_InteractSubscriber, BP_HUDSubscriber.
* Attach to NPCs/objects that need events.
* Character queries nearby/registered components and calls interface or direct functions.

Result: fewer global subscriptions, clearer ownership, easier lifetime management.

4. Gameplay Message Router (if you can use it)
   If you are on UE5 with Gameplay Message Subsystem / GameplayMessageRouter plugin available:

* Publish messages on Gameplay Tags.
* Subscribers listen only to tags they care about.
* Optionally include “Target Actor” in the message and filter.

Result: engineered for scalable decoupled messaging.

Why the “unique interface per subscriber” doesn’t scale

* Interface type proliferation (maintenance, compile time, cognitive load).
* Harder refactoring and versioning.
* Doesn’t solve addressing; it just bakes addressing into type count.

Minimal blueprint architecture (practical)

* BPI: BPI_GameEventReceiver

  * OnGameEvent(EventTag, Instigator, Target, PayloadStruct)
* Character holds:

  * Map<GameplayTag, ObjectReference> SubscribersByTag
* Subscriber registers/unregisters on BeginPlay/EndPlay.
* Character resolves and calls only the target.

If you want, describe your current “subscriber” types (HUD, NPC AnimBP proxy, interactables, etc.) and whether you already use Gameplay Tags; I can propose a concrete registry + payload struct layout that avoids casts and avoids broadcasts.
