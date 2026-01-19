## OpenAI ChatGPT v1.2026.006 UE.Gameplay.Plugins.Readiness.md

**Q: Are the unreal engine plugins Gameplay Message Subsystem / GameplayMessageRouter plugins production ready plugins?**

**A:** **Short answer:**
No. The **Gameplay Message Subsystem** and **GameplayMessageRouter** plugins originate from the **Lyra Starter Game sample**, not the core Unreal Engine distribution, and are **not officially documented/stable Epic-supported modules**. They can be used in a production project, but they are **not guaranteed “production-ready” by Epic** in the same way core engine subsystems (e.g., Gameplay Ability System) are. ([X157 Dev Notes][1])

**Details:**

* Both systems appear in the **Lyra sample project** as utility plugins for messaging/tag-based event broadcasting within local gameplay scope. They are **included in Lyra’s plugin folder**, and developers commonly copy them into their own projects for inter-object communication. ([X157 Dev Notes][1])
* **Epic does not list these in official UE plugin documentation** as fully supported engine modules with explicit stability guarantees; community and forum discussions indicate they are sample plugins with basic functionality, not core engine features. ([Epic Developer Community Forums][2])
* Packaging with **GameplayMessageRouter** generally works, but some users report issues during packaging without additional fixes; this suggests it’s not as thoroughly tested/stable as other built-in engine subsystems. ([Epic Developer Community Forums][3])
* The community has created **enhanced versions** of the router plugin to add features or address limitations, indicating the original may lack broader production-oriented refinements. ([github.com][4])

**Conclusion:** Use these plugins in production with caution. They are often functional and useful, but they are sample plugins lacking the same official support and stability guarantees as core engine systems. Advanced validation, testing, and possible modification may be required for robust commercial use.
```
[1]: https://x157.github.io/UE5/LyraStarterGame/Plugins/?utm_source=chatgpt.com "LyraStarterGame Plugins"
[2]: https://forums.unrealengine.com/t/thoughts-on-the-gameplay-message-subsystem/2359583?utm_source=chatgpt.com "Thoughts on the Gameplay Message Subsystem?"
[3]: https://forums.unrealengine.com/t/i-want-to-use-the-gameplaymessagerouter-plugin-from-lyra-in-my-project/1337495?utm_source=chatgpt.com "I want to use the \"GameplayMessageRouter\" plugin from ..."
[4]: https://github.com/akkz/UGCGameplayMessageRouter?utm_source=chatgpt.com "Gameplay Message Router (UGC Enhanced Version)"
```
**Q: Are there production ready third party versions?**

**A:** Yes, but “production ready” depends on vendor support cadence, engine-version coverage, and your platform/replication requirements.

Practical options

1. Commercial, vendor-supported (most “production-ready” posture)

* eelDev “Gameplay Messaging System” (Fab listing + vendor documentation + active product discussion thread). This is explicitly positioned as a project-wide, reference-free messaging layer similar in intent to Lyra’s router, with a vendor support surface (docs/forums) that is usually what teams mean by “production ready.” ([Fab.com][1])

2. Open-source, community-maintained (can be production-used, but you own risk)

* imnazake/gameplay-message-router: a standalone router-style plugin. Notes in its own README include “Network Replication Support: Not Supported,” and there are version-support issues reported (e.g., requests for higher UE versions). No formal releases are published, which typically implies extra integration burden. ([GitHub][2])
* akkz/UGCGameplayMessageRouter: an “enhanced” Lyra-derived variant aimed at UGC workflows; useful if you like the Lyra design but want additional features. Treat as community software (review maintenance activity/compat yourself). ([GitHub][3])

Selection criteria (quick filter)

* Engine coverage: verify explicit support for your exact UE minor (e.g., 5.3/5.4/5.5/5.6/5.7) and your target platforms.
* Replication: most router-style systems are local-only unless you wrap them with RPC/multicast; confirm explicitly (imnazake states no replication). ([GitHub][2])
* Packaging/shipping history: check for “packaged build runs” confirmation and reported issues (Lyra-derived router has had packaging/run reports in the wild). ([Epic Developer Community Forums][4])
* Maintenance signals: releases/tags, recent commits, issue response times, vendor forum presence.

If you state your UE version and whether you need network replication, I can narrow this to the best-fit option(s).
```
[1]: https://www.fab.com/it/listings/03b46864-12de-41c6-8624-00ece4d59e19?utm_source=chatgpt.com "Gameplay Messaging System"
[2]: https://github.com/imnazake/gameplay-message-router?utm_source=chatgpt.com "imnazake/gameplay-message-router"
[3]: https://github.com/akkz/UGCGameplayMessageRouter?utm_source=chatgpt.com "Gameplay Message Router (UGC Enhanced Version)"
[4]: https://forums.unrealengine.com/t/i-want-to-use-the-gameplaymessagerouter-plugin-from-lyra-in-my-project/1337495?utm_source=chatgpt.com "I want to use the \"GameplayMessageRouter\" plugin from ..."
```
