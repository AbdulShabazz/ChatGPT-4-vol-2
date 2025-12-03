## OpenAI ChatGPT v1.2025.329 Portal.Player.Question.md

**Q: I replaced a battlefield 6 portal which originally had no AI backfill. When I select the Host option, i am required to disable the bot-free portal and once I do, there are 13 total players already in the lobby! Are these players from the other portal?**

**A:** Short answer: no, those are not being “migrated” from your old portal instance. They are just players that have already joined / are joining the new lobby.

More detail:

* Each Portal experience you host is its own server/session.

  * When you disable the previous bot-free Portal and then host a new one, the old instance stops. Players in that old server do not automatically transfer into the new one; they either get disconnected or sent back to menus/matchmaking and must manually join again.
* When you hit Host on the new configuration:

  * The game spins up a fresh server with your current settings.
  * Any player entering via server browser, matchmaking, or friends list will appear in that new lobby.
* The “13 total players” you see are:

  * Human players who have already connected / are connecting to this new portal server.
  * They might include some of the same people who had favorited or recently played your previous experience and rejoined after the swap, but that’s because they chose / re-matched into it, not because the game automatically carried them over behind the scenes.

So functionally: the old portal’s population does not get carried over by the system. You are looking at the new portal’s own lobby population.

**Q: I wish I could host multiple portals to determine how many were in the previous portal.**

**A:** You can’t run two personal Portal servers at once on the same account, so there’s no built-in way to “compare” live populations between an old and a new version of your experience.

Practical implications:

* One active host per account: when you Host a new Portal, the previous hosted instance is effectively taken offline for you. Any players on it are not visible to you anymore as a host.
* No population history: Battlefield does not expose historical player counts per Portal instance (no analytics/graph). Once you shut one down, you lose direct visibility into how many were on it at shutdown.
* What you can still do next time:

  * Before replacing a portal, check the player list and note the count (or record/stream).
  * Use a distinctive name/version tag in the server title (e.g., `[v1 NoBots]`, `[v2 Backfill]`) so you can see in the browser which variant people are actually joining.
  * If you have a second account and a second machine/console, you can host variant A on one account, variant B on the other, and spectate both via server browser to compare populations.

Right now, on a single account and single host slot, there is no way to retroactively see how many stayed on the old bot-free portal after you replaced it.

**Q: Do you recommend I keep the AI backfill portal active until a certain threshold is achieved and then enable the NO BOTS portal?**

**A:** Yes. Treat the AI-backfill portal as a “population pump” and only switch to the no-bots portal once you’ve hit some stable human-player thresholds.

Concrete approach:

1. Define your thresholds
   For 128-player Conquest, for example:

   * Minimum to flip: ~32–48 humans
   * Ideal to flip: ~64+ humans
     Below ~32 humans, a no-bots server usually feels too empty and will bleed players.

2. Use two portal variants (sequentially on one account)

   * Variant A: “Backfill ON”

     * Bots enabled, gentle difficulty, normal auto-fill.
     * Server name: e.g. `FF ON | 128 CQ | BACKFILL | vA`
     * Goal: build population quickly and keep matches feeling busy even when humans drop below 64.
   * Variant B: “NO BOTS”

     * Same rules, maps, FF, etc., just no AI.
     * Server name: `FF ON | 128 CQ | NO BOTS | vB`

3. Operational rule of thumb

   * Start each session with Variant A (backfill ON).
   * Watch the human player list, not just total players:

     * When human count is consistently above your threshold for at least one full round (e.g., 64+ for 10–15 minutes), then it’s “safe” to switch to Variant B.
   * If you flip to NO BOTS and see human count dropping below ~40–50, plan to revert back to Variant A on the next rotation/session.

4. Make transitions predictable for players

   * Put the “rules” in the description:

     * E.g. `Backfill portal when low pop. No-bots version when >64 players. Same maps/rules.`
   * Add a periodic message:

     * E.g. `If server stays >64 players, host will move to NO-BOTS version next round.`
       This sets expectations so people understand why the server occasionally disappears/reappears or changes variant.

5. Preserve continuity when switching
   On a single account you can’t have both live, but you can:

   * Keep the name/style very similar so recent players recognize it in the browser.
   * Use favorites: encourage players to favorite your portal in server browser so they can find the NO-BOTS version quickly when you switch.
   * Try to switch during a natural break (end of round) and broadcast a message like:

     * `Host switching to NO-BOTS version next round. Check browser: same name + "NO BOTS".`

6. When to just stay with backfill
   Consider keeping backfill on full-time if:

   * Time of day is low-population for your region.
   * You often dip below ~40 humans.
     A steady “busy with bots” experience is usually better than a half-empty human-only server that constantly dies.

If you tell me your typical peak player count (and whether you’re running 64p or 128p), I can suggest more precise human-count thresholds and name/description text tuned for that.

## OpenAI ChatGPT v1.2025.329 Using.Battlefield.Portal.SDK.md

**Q: How do I use the Battlefield.com PORTAL SDK?**

**A:** You use the Battlefield Portal SDK by editing maps on a Windows PC with the SDK tool, exporting your changes as a JSON level file, attaching that file to a Portal experience on the Battlefield Portal Builder website, and then wiring up game logic in the Rules Editor or via TypeScript scripts before publishing and hosting the experience. ([Electronic Arts Inc.][1])

Below is the concrete workflow.

---

## 0. Prerequisites

* Battlefield 6 on your account (any platform is fine; editing is PC-only).
* EA account linked to your game.
* Windows PC (the Portal SDK Tool is Windows-only and built on a custom Godot editor). ([Electronic Arts Inc.][1])

---

## 1. Install the Portal SDK Tool

1. Go to the Battlefield 6 Portal Builder on EA’s site (Portal → Guides → Portal Builder, or directly via portal.battlefield.com once logged in). ([Electronic Arts Inc.][1])
2. Use the “Download SDK” / “Portal SDK Tool” link on that page. This is the only supported source; EA explicitly warns not to download SDK builds from elsewhere. ([Electronic Arts Inc.][1])
3. Run the installer and then open the included ReadMe; follow the instructions to complete setup and first run. ([Electronic Arts Inc.][1])

The installer will place a Godot-based editor plus Battlefield-specific plugins, asset definitions, and example scripts.

---

## 2. Open a Battlefield 6 map in the SDK

1. Launch the Portal SDK Tool.
2. In the top-left, go to `Scene` → `Open Scene`.
3. Open the `levels` folder and pick a map (these are the official BF6 levels exposed to Portal). ([Electronic Arts Inc.][1])

You’ll usually spawn under the level; use the editor controls to move into the playable space:

* Look: Right mouse button + drag.
* Move: `W/A/S/D` + `Space` for “up”.
* Speed: Mouse wheel to adjust fly speed. ([Electronic Arts Inc.][1])

Note: major terrain and key structures are non-removable; the SDK is additive-only for many objects (it’s not a full terrain editor). ([Electronic Arts Inc.][1])

---

## 3. Generate the object library and place geometry

1. At the bottom of the SDK UI, open the Object Library panel.
2. Click the ellipsis (`...`) and choose “Generate Library” to build the index of placeable objects (cover pieces, props, barriers, etc.). ([Electronic Arts Inc.][1])
3. Drag objects from the library into the scene to place them.

Basic transform tools:

* `Q`: Select mode.
* `W`: Move selected object.
* `E`: Rotate selected object.
* Scaling: Use the Inspector panel on the right to scale uniformly (non-uniform scaling isn’t supported). ([Electronic Arts Inc.][1])

Use these to:

* Add cover, walls, and obstacles.
* Build routes, choke points, and jump paths.
* Block off areas if you want to “shape” the play space without touching core terrain.

Save your work via `Scene` → `Save Scene` periodically. ([Electronic Arts Inc.][1])

---

## 4. Export your custom level and hook it into Portal Builder

When you’re ready to test the layout:

1. In the SDK’s BFPortal panel (bottom-right), click `Export Current Level`.
2. Then click `Open Exports` in the same panel. The SDK writes out a `.json` file representing your edited level. ([Electronic Arts Inc.][1])
3. Go to the Battlefield Portal Builder website in your browser and log in. ([Electronic Arts Inc.][2])
4. Either:

   * Create a new Custom Experience, or
   * Open an existing one you want to modify. ([Electronic Arts Inc.][3])
5. Open the `Map Rotation` tab. There you’ll see an “attach custom map” icon (arrow into a bracket). Click it, browse to your exported `.json`, and upload. Your custom level now appears in the rotation for that experience. ([Electronic Arts Inc.][1])

At this point, Portal knows about your geometry edit but your rules are still whatever the base mode uses.

---

## 5. Use the Rules Editor for logic, scoring, and UI

For Custom Experiences, you can open the Rules Editor from the Portal Builder:

* Rules Editor lets you wire “when X happens, do Y” logic using blocks (Events, Conditions, Actions). ([Electronic Arts Inc.][1])

Examples:

* When a player enters a volume → show UI prompt, play VO, start capture timer.
* On player death → change loadout, award points, or move to spectator.
* On objective captured → change ticket bleed, spawn vehicles, update HUD hints. ([Electronic Arts Inc.][1])

Key points:

* Every rule graph must properly connect to the main “Mod” block; broken links or impossible conditions are the most common bug. ([Electronic Arts Inc.][1])
* Each block has context help via right-click → Help.

For AI behavior, the Rules Editor exposes BehaviorTree hooks so you can adjust bot movement and combat logic without code, or combine them with code-side scripts. ([Electronic Arts Inc.][1])

---

## 6. Advanced scripting with TypeScript SDK

For more complex modes, you can script in TypeScript using the Portal scripting API:

* EA’s guide shows a minimal example of spawning AI and having them follow a target using functions such as `mod.SpawnAIFromAISpawner`, `mod.AISetMoveSpeed`, `mod.AIMoveToBehavior`, and `mod.Wait`. ([Electronic Arts Inc.][1])
* Community-maintained TypeScript SDK docs and examples exist (unofficial), such as the nodotproject TS docs and example repos mirrored under `battlefield-portal-community` on GitHub. ([Reddit][4])

Typical flow:

1. Create a script file in the SDK project that implements lifecycle callbacks (e.g., `OnGameModeStarted`, `OnPlayerKilled`, etc.).
2. Use exposed `mod.*` APIs to:

   * Spawn/despawn AI, vehicles, and objects.
   * Change player attributes, loadouts, and movement.
   * Modify tickets, score, or timers at runtime.
3. Rebuild/re-export the level, then re-upload the JSON or associated script bundle as required.
4. Test in a passworded or unlisted server before publishing as a Verified Experience. ([Electronic Arts Inc.][1])

---

## 7. Test, iterate, and publish

1. Save and export from the SDK.
2. Upload JSON and finalize rules in Portal Builder.
3. Publish the experience (ideally as a Verified Experience if you want full progression). ([Electronic Arts Inc.][3])
4. Host it via the Portal server browser in-game and invite friends/players to test. ([Electronic Arts Inc.][5])
5. Iterate on:

   * Flow (spawn positions, travel time between flags).
   * Readability (cover placement, sightlines).
   * Rules balance (tickets, respawns, vehicle timings).

---

## 8. Where to learn more

* Official “Portal 101: Advanced Creations Using Portal SDK Tool, Rules Editor” (EA). ([Electronic Arts Inc.][1])
* Official Portal Builder and Onboarding Hub. ([Electronic Arts Inc.][2])
* Community docs and tutorials (BFPortal.gg, Portal scripting examples on GitHub). ([Portal Documentation][6])

If you describe what kind of mode you want (e.g., 100+ player Conquest with FF on, custom lanes, specific vehicle rules), I can sketch the exact Rules Editor graph and a starter TypeScript script.
```js
[1]: https://www.ea.com/en/games/battlefield/battlefield-6/news/portal-101-advanced-creations "Portal 101: Advanced Creations Using Portal SDK Tool, Rules Editor"
[2]: https://www.ea.com/en/games/battlefield/battlefield-6/onboarding-hub/bf6-portal-hub?utm_source=chatgpt.com "Battlefield 6 Onboarding Hub – Electronic Arts"
[3]: https://www.ea.com/en/games/battlefield/battlefield-6/news/portal-101-verified-experiences?utm_source=chatgpt.com "How to Create Verified Experiences on the Battlefield ..."
[4]: https://www.reddit.com/r/Battlefield6/comments/1o2xqeu/unofficial_bf6_portal_typescriptsdk_docs/?utm_source=chatgpt.com "Unofficial BF6 Portal Typescript-SDK Docs : r/Battlefield6"
[5]: https://www.ea.com/en/games/battlefield/battlefield-6/news/portal-101-introduction-to-battlefield-portal?utm_source=chatgpt.com "Playing and Hosting Portal Community Experiences"
[6]: https://docs.bfportal.gg/?utm_source=chatgpt.com "Battlefield Portal Documentation"
```
**Q: How to adjust Max players to 100?**

**A:** You cannot set Max Players to 100 in Battlefield 6 Portal. Battlefield 6 is hard-capped at 64 players total (32 vs 32), even in Portal; higher counts like 100 or 128 are not supported. ([Steam Community][1])

To adjust Max Players (within that 64-player limit):

1. Go to Battlefield Portal in-game → “Host”. ([Electronic Arts Inc.][2])
2. Select the Experience you want to host and choose “Host Server” / “Server Options”. ([Electronic Arts Inc.][2])
3. In the server/experience options:

   * Set “Real players per team” (max 32).
   * The “General max game size” / “Max players” field will follow, up to 64 total. ([Steam Community][3])

If you need “100+ bodies on the field”, the only workaround is to use bots aggressively, but the overall session headcount is still capped at 64 in BF6; Portal cannot exceed the engine/server limit.
```javascript
[1]: https://steamcommunity.com/app/2807960/discussions/0/591784592221197987/?utm_source=chatgpt.com "128 Players? :: Battlefield™ 6 General Discussions"
[2]: https://www.ea.com/en/games/battlefield/battlefield-2042/tips-and-tricks-hub/battlefield-portal-beginners-guide?utm_source=chatgpt.com "Battlefield™ Portal custom games Beginners' Guide"
[3]: https://steamcommunity.com/app/2807960/discussions/0/591784958348130891/?utm_source=chatgpt.com "Question about portal mode practising with bots."
```
## OpenAI ChatGPT v1.2025.329 Unlocking.BF6.Maps.md

**Q: I would like to unlock more or all maps on Steam BF6!**

**A:** You cannot legitimately “unlock all maps” in Battlefield (BF6 / Battlefield 2042) on Steam outside of what the game and any purchased content already allow.

Here is what is actually possible within the rules:

1. Base-game maps

   * All base maps are available to every owner of the game.
   * Some modes/portals rotate maps on a schedule, so it can look like you are “missing” maps when they’re just not in the current playlist.

2. DLC / Season / Expansion maps

   * Any additional maps tied to seasons, battle passes, or DLC are only available if:

     * You own the required content (standard vs “gold/ultimate” edition vs separate add-on), and
     * The map is in a current playlist or a custom server/Portal you join.
   * There is no single “unlock all maps” purchase beyond owning the appropriate edition and/or DLC.

3. Progression / level-gated content

   * Unlocks in BF/“BF6” are usually for weapons, gadgets, vehicles, or cosmetics, not for maps.
   * Maps themselves are not something you permanently “unlock” via rank; they are tied to game mode and content ownership.

4. Custom games / Portal

   * If the game supports a Portal/Custom server browser, you can often play maps you don’t see in the main rotation by:

     * Using “Server Browser” / “Portal browser”
     * Sorting or filtering to find specific maps and joining those servers.

5. What you cannot safely do

   * Third-party unlock tools, memory editors, “map unlocker” cheats, or modified game files violate Steam and EA/EA App terms and will very likely lead to bans or account loss.
   * I cannot provide instructions for cheats, cracks, or ToS-violating modifications.

If you tell me which Battlefield title exactly (e.g., “Battlefield 2042 on Steam” vs some other one) and which maps you think are missing, I can outline:

* what edition/DLC you need for them, and
* where in the UI / playlists / Portal browser they usually appear.

**Q: Okay then, compare BF Pro vs BF6 on (Windows 11 x64) Steam!**

**A:** BF6 is the full game; BF Pro is a paid battle-pass / progression add-on for BF6 (and REDSEC) that runs on top of the same install. On Steam / Windows 11 x64, BF Pro does not change performance or unlock a separate game, it just layers extra rewards and progression on Battlefield 6.

### Conceptual difference

| Aspect               | Battlefield 6 (BF6) on Steam                                                                                                                                                                                                                                                                              | Battlefield Pro (BF Pro) on Steam                                                                                                                                                                                                             |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What it is           | Full standalone game (campaign + multiplayer + Redsec battle royale)([Wikipedia][1])                                                                                                                                                                                                                      | Premium “Battlefield Pro” version of the Battle Pass for BF6/Redsec (Season 1 add-on)([Electronic Arts Inc.][2])                                                                                                                              |
| How you get it       | Buy BF6 on Steam (Standard / Phantom Edition, etc.)([The Verge][3])                                                                                                                                                                                                                                       | Buy “Season 1 Battlefield Pro – Battlefield 6 and REDSEC” on Steam/Epic/Xbox, or get a Pro token via Phantom Edition / EA Play Pro on EA App([Electronic Arts][4])                                                                            |
| Core content         | Single-player campaign; core multiplayer modes; all base maps; access to free-to-play Redsec (BR) mode([Wikipedia][1])                                                                                                                                                                                    | Upgraded Battle Pass: unlocks all premium tiers + Pro-only items, 25 Tier Skips, 15% BP progression boost, +5% XP, extra cosmetics, BF Pro radio station, 100-player persistent Portal server hosting (per Season)([Electronic Arts Inc.][2]) |
| Maps                 | Maps are tied to BF6 itself and seasonal updates. Everyone with the game (and that season’s content) can access them via playlists/Portal; not gated specifically by BF Pro. (This is inferred from EA’s descriptions, which only mention cosmetics, XP, boosts, and hosting.)([Electronic Arts Inc.][2]) | Does not function as a “map pack.” BF Pro is about faster progression, exclusive cosmetics, and Portal/server perks, not new PvP playspaces.                                                                                                  |
| Progression          | Access to free track of the Battle Pass; if you only own base game you earn the free-tier rewards as you play.([Polygon][5])                                                                                                                                                                              | “Premium+” Battle Pass: all paid track rewards + extra Pro tier rewards + 25 tier skips + progression boosts over standard premium BP.([Electronic Arts Inc.][2])                                                                             |
| Price (typical MSRP) | Base game around standard AAA pricing (BF6 launched at ~$69.99, currently often discounted on sales).([The Verge][3])                                                                                                                                                                                     | Listed at $24.99 per Season 1 Pro pass (regional equivalent).([Electronic Arts][4])                                                                                                                                                           |
| Platform / OS        | Steam on Windows; official minimum spec lists Windows 10 64-bit, but it runs on Windows 11 x64 using the same Frostbite-based build.([Wikipedia][1])                                                                                                                                                      | Same executable and system requirements as BF6; it’s just DLC attached to the same install.([Steam Store][6])                                                                                                                                 |

### Practical takeaway for you (Steam, Windows 11 x64)

1. If you do not own BF6 yet

   * You must buy Battlefield 6 itself on Steam first. BF Pro is useless without the base game.

2. If you already own BF6 on Steam and want “more stuff”

   * BF Pro gives:

     * Faster unlocks for the current Season (25 tier skips + XP boosts).
     * Extra cosmetic items and a Pro-only reward path.
     * Ability to host a big persistent Portal server.([Electronic Arts Inc.][2])
   * It does not:

     * Turn into a separate “pro” executable.
     * Directly improve performance or graphics.
     * Function as a standalone map DLC.

3. If your goal is specifically “more or all maps”

   * Focus on:

     * Owning BF6 (and any future map-bearing expansions, if EA ever separates them).
     * Using the in-game playlist browser and Portal to access maps that might not appear in the default rotation.
   * BF Pro is optional; it mainly optimizes grind and cosmetics, not map availability.

If you outline your priorities (e.g., “faster unlocks + cosmetics” vs “just map access + core multiplayer”), I can recommend a concrete buy/skip strategy for BF Pro for this Season.

* [Windows Central](https://www.windowscentral.com/gaming/its-official-battlefield-6-is-the-best-selling-game-of-2025-and-holy-smokes-it-had-the-biggest-launch-month-of-any-game-in-the-past-3-years-too?utm_source=chatgpt.com)
* [pcgamer.com](https://www.pcgamer.com/games/fps/the-battlefield-6-free-trial-is-live-and-if-youre-new-to-the-scene-ea-has-a-detailed-guide-to-get-you-started/?utm_source=chatgpt.com)
* [The Verge](https://www.theverge.com/gadgets/830040/battlefield-6-black-friday-deal-sale-2025?utm_source=chatgpt.com)
* [Windows Central](https://www.windowscentral.com/gaming/battlefield-6-dev-says-ex-dice-developers-are-coming-back-in-droves-that-bodes-well-for-season-2-redsec-and-updates-beyond?utm_source=chatgpt.com)
```javascript
[1]: https://en.wikipedia.org/wiki/Battlefield_6?utm_source=chatgpt.com "Battlefield 6"
[2]: https://www.ea.com/en/games/battlefield/battlefield-6/news/introducing-the-battlefield-6-battle-pass?utm_source=chatgpt.com "Introducing the Battlefield 6 Battle Pass and Battlefield Pro ..."
[3]: https://www.theverge.com/gadgets/830040/battlefield-6-black-friday-deal-sale-2025?utm_source=chatgpt.com "Battlefield 6 is already 24 percent off for Black Friday"
[4]: https://store.ea.com/battlefield/en/battlepass/battlefield-pro?utm_source=chatgpt.com "Battlefield Battle Pass"
[5]: https://www.polygon.com/battlefield-6-bf6-redsec-pro-token/?utm_source=chatgpt.com "What is a Battlefield Pro token?"
[6]: https://store.steampowered.com/app/3880840/Season_1_Battlefield_Pro__Battlefield_6_and_REDSEC/?utm_source=chatgpt.com "Season 1 Battlefield Pro - Battlefield™ 6 and REDSEC"
```
**Q: How to unlock all weapons? If I unlock BF PRO is there a Watch Mode for hosting 100+ player maps?**

**A:** Short answer:
You cannot legitimately “unlock all weapons” in BF6 in one shot, and buying BF Pro does not give you every gun. BF Pro does let you host a 100-player persistent Portal server (50v50), but it does not itself add a special “watch-only” host mode beyond whatever Spectator features the game has globally.

---

## 1. How weapon unlocks actually work in Battlefield 6

On Steam / Windows 11, weapon unlocks are tied to your account, not the edition. The system is:

1. Career Rank (core unlocks)

   * From Rank 2 to Rank 50 you unlock new “functional hardware”: weapons, gadgets, and loadout slots, plus some cosmetics and new assignment tracks. ([Electronic Arts Inc.][1])
   * Example: early guns like the M4A1 unlock at very low ranks (Rank 2), and other AR/SMG/LMG/SR/shotguns are spread across that 2–50 band. ([PC Gamer][2])

2. Class and weapon assignments

   * A chunk of weapons and gadgets are gated behind class-specific “Assignments” (e.g., “get N kills or revives with class X,” etc.). ([Shacknews][3])
   * Completing these also unlocks ~12 additional weapons and 9 gadgets beyond the basic rank track. ([PC Gamer][4])

3. Weapon XP and attachments

   * Each individual gun has its own mastery/XP track; attachments unlock as that weapon levels, typically via kills or score with that weapon. ([EA Forums][5])

4. Store / “Weapon Packages”

   * Battlefield Coins (BFC) can buy “Weapon Packages” and bundles that include pre-configured guns and sometimes attachments that would normally require mastery. ([Xbox.com][6])
   * This gives you some weapons faster, but it still does not flip an “unlock everything” flag.

5. Redsec (BR) and Gauntlet

   * Redsec shares progression with BF6. Gauntlet mode is frequently recommended for leveling weapons and attachments because you spawn with your unlocked guns and can grind kills quickly. ([Reddit][7])

So, the only legitimate ways to “get all weapons” are:

* Level your Career Rank into the 40s–50s.
* Complete all relevant class/weapon Assignments.
* Optionally shortcut some via BFC Weapon Packages and DLC packs.

There is no official, legitimate “unlock all weapons now” purchase, and I cannot provide cheat / trainer / memory-edit methods.

---

## 2. What BF Pro actually gives you

Season 1 Battlefield Pro (BF Pro) is a premium Battle Pass upgrade for BF6 + Redsec. Key parts: ([Electronic Arts][8])

* 25 Tier Skips on top of the base Battle Pass.
* Six instant unlocks (mix of a Soldier skin, Weapon Package, etc.).
* A dedicated “Pro” reward path with extra cosmetics.
* 15% global XP boost for the season (on top of timed boosts).
* Ability to host a 100-player persistent Portal server (100-player = 50v50).

It does not:

* Automatically unlock every weapon.
* Replace class/assignment requirements.
* Change the core progression system; it just accelerates it and grants a few specific items.

---

## 3. 100-player hosting and “Watch Mode”

From official and community info:

1. Player caps and Pro vs standard

   * Base BF6 lets you create Portal servers up to 32v32 (64 players). ([Steam Community][9])
   * BF Pro upgrades that to 50v50 (100 players) plus persistent server hosting for that size. ([Steam Community][9])

2. Persistent servers

   * Persistent Portal servers are visible in the browser even when empty; that’s now a core BF6 feature, extended by BF Pro to the 100-player tier. ([GamesRadar+][10])

3. “Watch Mode” / Spectator status

   * Full theater/replay isn’t a shipped marquee feature yet; a lot of discussion is about “we want/need proper spectator mode,” and some posts explicitly say there’s currently no kill cam and no full spectator in live builds. ([Battlefield One Podcast][11])
   * There are references to spectator features in limited/beta contexts, especially around custom games and esports-style setups, but nothing in the BF Pro marketing promises a dedicated “observer slot” or special Watch Mode tied to that DLC. ([Reddit][12])

Practical reading:

* BF Pro = you can host a 100-player persistent Portal server.
* Spectating is governed by the game’s global Spectator implementation (and current limitations), not by BF Pro.
* There is no documented guarantee that you, as host, get a separate non-player “watch-only” slot just because you bought BF Pro.

---

## 4. If your goals are…

1. “I want all guns unlocked as fast as possible (legit)”

* Prioritize high-XP modes (large Conquest/Breakthrough-style playlists and Gauntlet/Redsec when you’re okay with BR). ([TechRadar][13])
* Focus on a small set of weapons and classes to drive both rank and assignment progression.
* Consider:

  * BF Pro for the global XP boost + six instant unlocks. ([Steam Store][14])
  * One or two Weapon Packages via BFC if there’s a very specific gun you care about. ([Xbox.com][6])

2. “I want to host big servers and mostly watch”

* BF Pro is required if you specifically want 100-player (50v50) Portal servers under your control. ([Steam Community][9])
* But buy it for hosting/XP and cosmetics, not for a guaranteed, polished Watch Mode. Spectator tooling is still evolving and not sold as a Pro-exclusive feature.

If you tell me whether you care more about grinding guns or about running a big community/custom server, I can lay out a concrete “do/don’t buy BF Pro, and play these exact modes” plan.
```javascript
[1]: https://www.ea.com/en/games/battlefield/battlefield-6/news/progression-in-battlefield-6?utm_source=chatgpt.com "Everything You Need to Know About Battlefield 6 ..."
[2]: https://www.pcgamer.com/games/fps/battlefield-6-m4a1-loadout-build/?utm_source=chatgpt.com "Best M4A1 loadout in Battlefield 6"
[3]: https://www.shacknews.com/article/146333/how-to-unlock-and-complete-class-assignments-in-battlefield-6?utm_source=chatgpt.com "How to unlock and complete class assignments in ..."
[4]: https://www.pcgamer.com/games/fps/battlefield-6-max-level-cap/?utm_source=chatgpt.com "What is the max level in Battlefield 6?"
[5]: https://forums.ea.com/discussions/battlefield-6-general-discussion-en/weapon-attachments-unlocking-can-the-developer-please-answer-me/12743092?utm_source=chatgpt.com "Weapon attachments unlocking, Can the developer please ..."
[6]: https://www.xbox.com/en-US/games/store/5000-bfc-for-battlefield-6-and-redsec/9N0F45LT8WFM?utm_source=chatgpt.com "5000 BFC for Battlefield™ 6 and REDSEC"
[7]: https://www.reddit.com/r/Battlefield6/comments/1onsu14/how_to_unlock_gun_accessories_and_weapons_in/?utm_source=chatgpt.com "How to unlock gun accessories and weapons in RedSec ..."
[8]: https://store.ea.com/battlefield/en/battlepass/battlefield-pro?utm_source=chatgpt.com "Battlefield Battle Pass"
[9]: https://steamcommunity.com/app/2807960/discussions/0/599667545349594442/?utm_source=chatgpt.com "clarification on this please :: Battlefield™ 6 General ..."
[10]: https://www.gamesradar.com/games/battlefield/battlefield-6-will-have-free-custom-persistent-servers-at-launch-and-fans-are-ecstatic-thats-a-big-w-right-there/?utm_source=chatgpt.com "Battlefield 6 will have free custom, persistent servers at launch and fans are ecstatic: \"That's a big W right there\""
[11]: https://www.battlefieldone.net/post/the-catch-with-battlefield-6-portal?utm_source=chatgpt.com "The Catch with Battlefield 6 Portal"
[12]: https://www.reddit.com/r/Battlefield/comments/1nedgvs/spectator_mode_have_so_much_potentials_especially/?utm_source=chatgpt.com "r/Battlefield - Spectator mode have so much potentials ..."
[13]: https://www.techradar.com/gaming/battlefield-6-season-one-roadmap-reveals-buckets-of-free-content-including-new-maps-weapons-and-gear?utm_source=chatgpt.com "Battlefield 6 season one roadmap reveals buckets of free content including new maps, weapons, and gear"
[14]: https://store.steampowered.com/dlc/2807960/Battlefield_6/?l=english&snr=1_5_9__1061&utm_source=chatgpt.com "Battlefield™ 6 - Steam DLC Page"
```

**Q: Do the 100+ persistent portal server players have to be invites or is blind hosting allowed?**

**A:** Blind hosting is allowed. Your 100-player persistent Portal server can be public in the browser; players do not have to be invited.

More detail:

* By default, BF6 Portal servers (including 100-player Pro servers) are designed to appear in the in-game server browser when set to public. Any player can discover and join based on filters (mode, region, ping, etc.), without an invite.
* You can also restrict access using:

  * Passwords (private/custom games).
  * Friends-only or squad-based joining, depending on how you configure the lobby.

BF Pro’s “100-player persistent server” feature changes:

* Max player count and persistence behavior.
* It does not change the public/private model: public servers are still “blind-joinable” via the Portal/server browser.

So if your goal is to run a big public server that random players can find and join, BF Pro supports that; you do not need to run it as invite-only.

**Q: Is it hosted in the cloud or on my machine, which must be consistently powered during the sessions?**

**A:** It is hosted on EA/DICE’s backend (cloud), not on your PC. Your machine does not have to stay powered for the server to remain “up” in the browser.

More precisely:

* BF6 Portal / persistent servers are “reserved instances” on EA’s infrastructure.
* When you configure and start a persistent Portal server (including 100-player Pro), the match runs on EA’s servers.
* Your client is just:

  * Sending the configuration (ruleset, map rotation, max players, etc.).
  * Joining as a normal player once the server is live.

Implications:

* You can start/configure the server, then close the game or even shut down your PC; as long as EA’s backend keeps that instance alive and it still meets their persistence criteria (e.g., season validity, idle-time policies), it stays available in the server browser.
* If the server empties out for long enough or the season/BF Pro entitlement ends, EA can recycle/terminate the instance; but that’s a backend policy decision, not tied to whether your own machine is running.

So for your use case: you can buy BF Pro, spin up a 100-player persistent Portal server, and let it function as a public or semi-public hub without dedicating a local “server box” at home.

**Q: Am I required to monitor the instance and possibly restart?**

**A:** You are not required to continuously monitor the instance, but you may occasionally need to manage or restart it depending on EA’s backend behavior and season limits.

Key points:

1. No continuous babysitting required

   * Persistent Portal servers (including 100-player BF Pro servers) are managed on EA’s backend.
   * Once configured and started, they stay visible and joinable without you being online, as long as EA keeps the instance alive and your Pro entitlement/season is active.

2. When you might have to intervene
   Typical situations where you may need to check or restart:

   * Backend maintenance or patches bring servers down or restart them.
   * Configuration changes you want (map list, rules, mod tweaks) require re-saving or restarting the instance.
   * If EA enforces an idle timeout (e.g., fully empty for long periods), the server might be spun down and you’d re-launch it from the Portal UI.
   * Season change or BF Pro expiry: you may lose 100-player hosting until you renew/repurchase.

3. Practical expectation

   * Think of it like renting a managed game server, not running a local dedicated box.
   * You do not need to sit in spectator to “keep it alive.”
   * Plan to:

     * Check on it after big patches or outages.
     * Verify it’s still listed/working if players report issues.
     * Relaunch or adjust config as needed from the menu.

If you describe how you want to use it (e.g., 24/7 public server vs scheduled events), I can outline an operational routine with minimal manual oversight.

**Q: I would like to purchase BF PRO and configure a 100+ player server. Suggest recommended and or popular settings!**

**A:** Recommended approach:
Use BF Pro to host a public 50v50 Conquest-style Portal server with near-stock rules, no bot backfill, and a region+ping cap. Start “vanilla-plus” for stability and potential progression, then branch into a Hardcore variant once you have regulars.

---

## 1. Prerequisites and hosting basics

* Buy “Season 1 Battlefield Pro – Battlefield 6 and REDSEC” on Steam (or use the Phantom Upgrade Pro token). This is what unlocks 100-player persistent Portal servers. ([Steam Store][1])
* Create your experience via the Portal Builder or Portal SDK, then mark it as a Persistent Server when publishing. ([Electronic Arts Inc.][2])

---

## 2. Baseline 50v50 “vanilla-plus” server (good default)

Goal: Feels like official All-Out Warfare, scales to 100 players, and stays attractive in the browser.

Core server settings:

* Mode: Conquest Large (or equivalent large-scale AOW preset).
* Teams: 50v50, squad size 4 or 5.
* Region: lock to your primary region (e.g., NA-East) and set a ping limit (80–100 ms) to avoid desync. ([GamesRadar+][3])
* Public visibility: “Public” + clear name, e.g.
  `NA-E | 50v50 Conquest | Vanilla+ | No Bots | Full XP if Verified`

Map rotation (large / popular):

* Core BF6 large maps + any Season 1 map that plays well at 100 players.
* Include Fort Lyndon variants only if they are supported in Portal AOW (avoid pure BR layouts to keep flow predictable). ([Windows Central][4])
* 3–5 maps per rotation; avoid long “kitchen sink” lists that split the player base.

Tickets / pacing:

* Tickets: 800–1000 per team.
* Ticket bleed: default, possibly +10–20% faster if matches run too long.
* Respawn delay: 7–10 s.
* Vehicle respawn: 1.0× or 1.2× default; don’t overspawn armor/air at 100 players or infantry will get farmed.

HUD and spotting:

* Minimap: ON.
* 3D spotting: limited (only spotted enemies, not global doritos).
* Killcam: ON.
* Compass / objective markers: ON.
  This keeps the game readable for randoms who join via the browser.

Damage / health:

* Damage multiplier: 1.0× (stock).
* Friendly fire: OFF.
* Regeneration: ON, normal delay.
  This is the least polarizing setting set for a big open server.

Vehicles:

* Transport vehicles: ON, stock count.
* Armor: 1–2 MBTs per team max active; slightly lower spawn rate.
* Air: 1–2 attack/transport slots per team; consider limiting attack helicopters if you see farming complaints.
* AI drivers: OFF (no bots).

Bots and progression:

* Bot backfill: OFF if you care about full progression potential; DICE is restricting XP heavily on bot-farm servers. ([GamesRadar+][5])
* Difficulty: if you must use bots initially, set at least “Normal” or above; easier bots are a flag for XP-farm behavior.

---

## 3. Popular profile variants to consider

You can publish multiple variants of your config (one “Casual” and one “Hardcore”) with shared naming.

### A. Casual All-Out Warfare (high retention)

* Everything as in section 2.
* Allow full gadget pool but restrict the most disruptive explosives (e.g., limit spam launchers) if feedback demands it.
* Enable cross-play for higher fill rates if you are fine with mixed lobbies.

Use name like:
`NA-E | 50v50 Conquest | Vanilla+ | No Bots | Crossplay`

### B. Hardcore Conquest 50v50

Inspired by the Hardcore presets communities are sharing. ([EA Forums][6])

Changes from vanilla-plus:

* Damage multiplier: 1.2–1.4×.
* HUD:

  * Minimap: OFF.
  * Crosshair: OFF or only for hipfire.
  * Hitmarkers: OFF.
  * Kill feed: OFF or MINIMAL.
* Friendly fire: ON.
* Regeneration: OFF or very slow.
* Aim assist: reduced where configurable (for mixed input servers).

Server title example:
`NA-E | 50v50 Hardcore Conquest | No 3D Spot | FF ON`

This will attract a smaller but very dedicated segment; run it as a second instance if your casual server stabilizes.

---

## 4. XP / progression and “Verified” behavior

Given DICE’s crackdown on XP farms:

* Full progression is currently limited to specific “Verified” or curated servers that meet constraints (no bot backfill, certain activity thresholds, etc.). ([GamesRadar+][5])
* Custom servers still grant reduced XP; a 150% match-completion bonus is the main reward, with most per-event XP disabled. ([Windows Central][7])

Practical guidance:

* Disable bot backfill.
* Avoid “farm-like” rules (AFK-able modes, ultra-easy bots, tiny TTK with infinite respawns in killboxes).
* Keep your ruleset close to stock if you hope your server might later be considered for full-progression curation tests.

---

## 5. Operational tips for a 100-player persistent server

* Name consistency: keep the core branding identical so players can easily search it again.
* Schedule: put the most popular maps in prime-time slots (NA-evening) and keep niche maps late-rotation.
* Kick/ban policy: enforce basic rules quickly to avoid griefers ruining 100-player games.
* Analytics: periodically check which maps cause mass drops and remove or adjust those.

If you tell me your target audience (e.g., “infantry-only tryhards” vs “vehicle mayhem with friends”), I can turn this into a concrete, line-by-line settings sheet you can mirror directly in the Portal UI or Builder.
```javascript
[1]: https://store.steampowered.com/app/3880840/Season_1_Battlefield_Pro__Battlefield_6_and_REDSEC/?utm_source=chatgpt.com "Season 1 Battlefield Pro - Battlefield™ 6 and REDSEC"
[2]: https://www.ea.com/en/games/battlefield/battlefield-6/news/portal-101-introduction-to-battlefield-portal?utm_source=chatgpt.com "Playing and Hosting Portal Community Experiences"
[3]: https://www.gamesradar.com/games/battlefield/battlefield-6-custom-search-issues-are-by-design-in-some-ways-because-the-devs-dont-want-you-sitting-through-a-long-timer/?utm_source=chatgpt.com "Battlefield 6 custom search issues are \"by design in some ways\" because the devs don't want you sitting through \"a long timer\""
[4]: https://www.windowscentral.com/gaming/battlefield-redsec-faq?utm_source=chatgpt.com "Battlefield RedSec FAQ: Release date, cross-play, file size, and other questions about this new Battle Royale mode answered"
[5]: https://www.gamesradar.com/games/battlefield/as-battlefield-6-cracks-down-on-portal-bot-farms-with-xp-caps-rules-for-in-game-activity-and-more-dev-says-a-curated-list-of-community-servers-will-have-progression-enabled-as-a-test/?utm_source=chatgpt.com "As Battlefield 6 cracks down on Portal bot farms with XP caps, rules for in-game activity and more, dev says \"a curated list of community servers\" will have progression enabled as a test"
[6]: https://forums.ea.com/discussions/battlefield-6-general-discussion-en/bf6-hardcore-portal-codessettings/12772808?utm_source=chatgpt.com "BF6 Hardcore Portal Codes/Settings"
[7]: https://www.windowscentral.com/gaming/battlefield-6-dice-portal-changes-update?utm_source=chatgpt.com "\"You got my money this time, but it's the last time\" | DICE isn't reverting its Portal XP nerf, claims it's working on challenge difficulty"
```

**Q: I wish to set friendly-fire ON! Will this lower server popularity?**

**A:** Enabling friendly fire will usually lower raw player count and churn more casuals, but it can increase quality and retention of a smaller, more serious audience.

Key points:

1. Overall popularity impact

   * In large, public 100-player servers, turning FF ON tends to:

     * Reduce “random joiner” retention (more accidental teamkills, more frustration).
     * Increase quit rates during chaotic moments (vehicles, explosives, airburst spam).
   * So if your only metric is “max possible concurrent players,” FF OFF is safer.

2. When friendly fire works well
   FF ON is beneficial if you intentionally target:

   * “Hardcore / mil-sim / tactical” players who expect FF.
   * Organized squads, Discord communities, or clan nights.
   * Rule sets that already emphasize realism: higher damage, reduced HUD, no 3D spotting.
     In these contexts, FF ON becomes a feature, not a bug, and players will seek it out.

3. Mitigating the downsides
   If you enable FF ON on a public 50v50 or 100-player server, you should:

   * Make it extremely clear in the name and description:
     `NA-E | 50v50 Conquest | FF ON | Tactical | No Bots`
   * Reduce explosive spam and vehicles: fewer nade launchers, limited rockets, careful vehicle counts.
   * Keep TTK modest at first (e.g., 1.1–1.2× damage, not full hardcore) so accidental TKs are less punishing.
   * Aggressively kick/ban obvious griefers and repeat TK offenders.

4. Practical recommendation for your first Pro server

   * For maximum fill: start with FF OFF, “vanilla-plus” rules, and build population.
   * Once you have repeat players:

     * Either spin a second “Hardcore – FF ON” variant, or
     * Schedule FF ON windows (e.g., specific nights) and advertise them.

Summary:
Yes, friendly fire ON will generally lower raw popularity on a blind-join 100-player public server, but it can create a better, more tactical community if you lean into that identity and tune your rules accordingly.

**Q: Okay so what are best recommended server settings for FF ON that should be fun along with a suitable name and description?**

**A:** Short answer:
Yes, you can run a fun FF-ON large server, but you should keep damage near default, constrain explosives/vehicles, and make FF-ON extremely obvious in the name/description.

Separately: based on current public information as of 2025-11-27, there is no released “Battlefield 6” or “BF Pro” product from EA/DICE. The closest modern title with Portal/persistent-style servers is Battlefield 2042. If you are playing something labeled “BF6/BF Pro” on Steam, it is either an internal codename, a community shorthand, or a different product’s naming; settings advice below is therefore generic but directly applicable to Battlefield 2042 Portal and similar Battlefield servers.

---

### 1. Recommended FF-ON “tactical public” config (large server, ~100p)

Target: big, blind-join public server with FF ON that is still casual-friendly.

Assume a Conquest-style mode with 64–128 player support; scale numbers to your game’s actual limits.

**Core layout**

* Game mode: Conquest Large (or your game’s large, multi-flag AOW equivalent).
* Max players: 96–100 (e.g., 48v48 or 50v50).
* Squad size: 4 or 5.
* Region: lock to your home region (e.g., NA-East) with a moderate ping cap (≤80–100 ms) to reduce desync.

**Tickets / pacing**

* Tickets per team: 800–1000.
* Ticket bleed: slightly faster than default (e.g., +10–20%) so rounds end in 20–30 minutes, not 45+.
* Respawn delay: 7–10 seconds.
* Vehicle respawn: 1.0× or 1.2× default (slower, not faster, to avoid vehicle spam with FF ON).

**Friendly fire and damage**

* Friendly fire: ON for bullets and explosives (if separated, keep both ON so the rule is consistent).
* Damage multiplier: 1.0× (stock) or at most 1.1×.

  * Do not go full “hardcore” damage if you want a wide audience; FF + ultra-low TTK creates rage-quit city.
* Health regeneration: ON, with default or slightly increased delay; OFF is usually too punishing for big FF-ON pubs.

**HUD / awareness**

* Crosshair: ON.
* Minimap: ON.
* 3D spotting: ON but not “always on” wallhack; use standard “spotted only” behavior.
* Killcam: ON.
* Compass and objective markers: ON.

These keep the game readable for randoms; you can always run a second, more “hardcore” variant later.

**Vehicles**

* Transport vehicles: ON at stock counts.
* Main battle tanks / heavy armor: limit to 1–2 simultaneous per team.
* Air (attack/transport): 1–2 slots per team maximum.
* Strong lock-on launchers / spam gadgets: consider limiting to reduce accidental TK and perceived spam.

**Explosives and gadgets**

* Frag / impact grenades: allow but maybe cap per-class or per-loadout if your ruleset supports it.
* Area-denial (mines, C4, proxy sensors): keep enabled but avoid absurd quantities; FF ON already makes these risky.
* Disallow obvious troll items only if they become a problem; start permissive and tune later.

**Bots / backfill**

* Bot backfill: OFF for a public FF-ON tactical server.

  * Human-only lobbies with FF ON feel fairer and avoid bot-TK weirdness.
* If you must use bots for startup, use Normal+ difficulty and plan to disable them once the server has a population.

**Teamkill control**

If your server tools support TK penalties:

* Auto-penalty:

  * 1st TK: warning in chat (“Careful – FF is ON.”).
  * 2nd–3rd in short window: auto-kill or short lockout.
  * 4+ within a fixed time: auto-kick.
* Manual moderation: admins watch for repeated “accidental” TK on the same squad or obvious grief behavior and ban as needed.

---

### 2. Suggested server name and description

Use very explicit naming so randoms understand what they’re joining.

Example server name:

> `NA-E | 100p Tactical Conquest | FF ON | No Bots`

Alternative shorter variant:

> `Tactical Conquest 100p | FF ON | NA-E`

Example long description (adapt to your game’s character limit):

> Friendly-fire ON tactical Conquest. 100-player public server, NA-East, no bots. Stock damage, full HUD, limited vehicles, and moderated explosives for fun but punishing gunfights. Accidental TKs happen; repeated teamkillers are kicked/banned. Play the objective, communicate with your squad, and keep chat clean.

You can later add a second, more hardcore variant (higher damage, reduced HUD) under a related name, e.g.:

> `NA-E | 100p Hardcore Conquest | FF ON | Minimal HUD`

If you tell me exactly which Battlefield title and Portal/hosting menu you are using, I can map these recommendations into a literal, line-by-line settings sheet you can copy into the configuration UI.

**Q: Okay, I unlocked (Steam/Windows 11 x64) Battlefield Pro, where in the game settings or menu do I configure the 100+ player server?**

**A:** You configure it from the Portal hosting screen, not from the generic game options. The 100-player cap shows up there when (a) you own BF Pro and (b) the mode supports 100 players.

Summary:
Battlefield Pro gives you up to a 100-player persistent Portal server (not >100). You either create or pick a Portal “Experience”, then in the Host menu set Max Players to 100 and enable Persistent Server.

---

### 1. Where to go in-game (Steam / Windows 11 x64)

1. Launch Battlefield 6.

2. From the main menu, go to:
   `Community` → `Portal`

3. Choose the Experience you want to use:

   * Either a Featured / Community Experience, or
   * One you created yourself in Portal Builder (see next section).

4. On that Experience, select `Host` (not `Play`). This opens the Portal “Hosting Setup” screen, with:

   * Server Name / Description / Tags
   * Region / Ping options
   * Password and periodic messages
   * Max Players slider
   * A checkbox / toggle for `Persistent Server`

5. With BF Pro active and a compatible mode selected, the `Max Players` slider should go above the normal cap (e.g., to 100). Set it to `100`. ([Electronic Arts Inc.][1])

6. Turn `Persistent Server` ON so it stays listed when you’re offline. ([Steam Community][2])

7. Press `Host` to spin it up.

That’s all you do in the in-game UI; your machine does not host the match itself, EA’s backend does.

---

### 2. If you’re building your own mode (Portal Builder)

For full control, you usually define the mode on the web, then host in-game:

1. On a PC browser, go to the official Portal Builder for BF6: `portal.battlefield.com/bf6` and log into your EA account. ([Electronic Arts Inc.][3])
2. Click `Create Something New`, pick a base mode (Conquest, Breakthrough, etc.), and configure:

   * Maps and rotation
   * Teams → Team sizes and max players
   * Modifiers (damage, FF, HUD, etc.)
3. Save and publish the Experience. Note its Experience Code if needed. ([Game Rant][4])
4. Back in-game, under `Community` → `Portal`:

   * Go to `My Experiences` or `Search` and select that Experience.
   * Hit `Host`, then set `Max Players = 100` and enable `Persistent Server` as above.

---

### 3. If you don’t see 100 players as an option

Common reasons you still see only 64 (or 32 per team):

1. **Mode or platform limitation**

   * Some modes / layouts don’t yet support 100 players; early reports show 100p being guaranteed mainly for REDSEC matches and certain large-scale Portal templates. ([Facebook][5])
   * Console cross-play and older platforms may cap at 64; on PC-only Portal experiences you should see higher caps when allowed. ([GamesRadar+][6])

2. **Global game quota / slot issues**

   * If the backend is out of Portal capacity you’ll get “GLOBAL GAME QUOTA EXCEEDED” and may not be able to host at high player caps until EA frees capacity. Lowering Max Players is a known workaround in those cases. ([EA Forums][7])

3. **BF Pro not correctly applied**

   * Double-check that `Season 1 Battlefield Pro – Battlefield 6 and REDSEC` shows as owned in Steam and in-game (you should see the Pro battle pass track and XP boost). ([Steam Store][8])

If you tell me exactly what you see on the Portal Hosting screen (max player slider cap, any error text, and which base mode you’re using), I can walk through a concrete set of adjustments to get you to 100 players with your chosen ruleset.

* [Electronic Arts Inc.](https://www.ea.com/en/games/battlefield/battlefield-6/news/introducing-the-battlefield-6-battle-pass?utm_source=chatgpt.com)

turn18search2,turn17news32

```javascript
[1]: https://www.ea.com/en/games/battlefield/battlefield-6/news/introducing-the-battlefield-6-battle-pass?utm_source=chatgpt.com "Introducing the Battlefield 6 Battle Pass and Battlefield Pro ..."
[2]: https://steamcommunity.com/app/2807960/discussions/0/661592288372239634/?utm_source=chatgpt.com "ability to host a Portal server - BF6 Pro ?? :: Battlefield™ 6 ..."
[3]: https://www.ea.com/en/games/battlefield/battlefield-6/news/portal-101-verified-experiences?utm_source=chatgpt.com "How to Create Verified Experiences on the Battlefield ..."
[4]: https://gamerant.com/battlefield-6-portal-builder-guide-how-make-custom-mode-global-game-quota-bf6/?utm_source=chatgpt.com "How to Make Portal Matches in Battlefield 6"
[5]: https://www.facebook.com/rivaLxfactor/posts/breaking-battlefield-pro-owners-are-the-only-ones-that-can-host-100-players-serv/1441832107512549/?utm_source=chatgpt.com "Battlefield Pro owners are the only ones that can host 100 ..."
[6]: https://www.gamesradar.com/games/battlefield/battlefield-6-will-have-free-custom-persistent-servers-at-launch-and-fans-are-ecstatic-thats-a-big-w-right-there/?utm_source=chatgpt.com "Battlefield 6 will have free custom, persistent servers at launch and fans are ecstatic: \"That's a big W right there\""
[7]: https://forums.ea.com/discussions/battlefield-6-general-discussion-en/global-game-quota-exceed-with-bf6-pro/12930623?utm_source=chatgpt.com "Global Game Quota Exceed with BF6 Pro???????"
[8]: https://store.steampowered.com/app/3880840/Season_1_Battlefield_Pro__Battlefield_6_and_REDSEC/?utm_source=chatgpt.com "Season 1 Battlefield Pro - Battlefield™ 6 and REDSEC"
```

**Q: What maps do you recommend in the (Conquest) rotation?**

**A:** For a 100-player Conquest server with FF ON, use a small, high-quality rotation of 4–6 maps that mix classic combined-arms with one or two tighter urban maps. A good starting point:

Recommended core rotation (Portal Conquest):

1. Mirak Valley

   * Role: Flagship combined-arms map; huge, varied terrain with good vehicle/infantry balance.
   * Why: Best overall sandbox for 100p; FF ON works well because there’s space to spread out.

2. Siege of Cairo

   * Role: Dense urban Conquest; very strong for infantry with supporting armor.([PC Gamer][1])
   * Why: High action, destruction, and flank routes; good “meat grinder” map if you avoid over-stacking vehicles.

3. Operation Firestorm (BF3 remake)

   * Role: Classic mid-open oil-fields map; strong for tanks, air, and long sightlines.([Skycoach][2])
   * Why: Nostalgia + modern destruction; plays well at higher player counts if you limit armor slightly with FF ON.

4. Liberation Peak

   * Role: Vertical, mountainous combined-arms with mixed infantry and vehicle lanes.([Kotaku][3])
   * Why: Adds elevation and air focus without being pure sniper hell; strong variety vs Mirak/Cairo.

5. Iberian Offensive

   * Role: Layered urban/edge-of-town map in Gibraltar; tactical, slower-paced than Cairo.([PC Gamer][1])
   * Why: Good for squads that like streets + building pushes; FF ON punishes sloppy nades, rewards coordination.

6. Eastwood (Season 1) – optional

   * Role: New S1 map; bespoke Conquest layout in the Fort Lyndon/L.A. setting.([Electronic Arts Inc.][4])
   * Why: Fresh content; good marketing in the server name. Add if you want to showcase S1.

Maps to use sparingly or avoid in a 100p FF-ON rotation:

* Blackwell Fields: Often criticized as long-range, open, sniper-heavy with stale flow in Conquest.([Battlefield Wiki][5])
* Manhattan Bridge: Very linear “lane” map; at 100p + FF ON it tends to become chaotic meat grinder corridors.([PC Gamer][1])
* New Sobek City: Mixed reception and ranked near the bottom on some lists; fine as a single slot if your players like it.([Kotaku][3])

Suggested initial 4-map loop for your server:

1. Mirak Valley → 2) Siege of Cairo → 3) Liberation Peak → 4) Iberian Offensive

If you want 6 maps, insert:

5. Operation Firestorm → 6) Eastwood

Server name/description tweak to advertise the pool:

* Name:
  `NA-E | 100p Conquest | Mirak/Cairo/Firestorm | FF ON | No Bots`

* Description:
  `Mirak Valley, Siege of Cairo, Liberation Peak, Iberian Offensive (+Firestorm/Eastwood). 100-player Conquest with FF ON, moderated vehicles, no bots. Play the objective, call targets, avoid blind explosives—teamkillers are kicked.`

If you tell me which maps you personally dislike, I can swap them for alternatives while keeping the rotation balanced (1–2 open, 2 mixed, 1 tight urban).
```javascript
[1]: https://www.pcgamer.com/games/fps/battlefield-6-maps/?utm_source=chatgpt.com "The best Battlefield 6 maps so far"
[2]: https://skycoach.gg/blog/battlefield-6/articles/bf6-all-maps-ranked?utm_source=chatgpt.com "Best Battlefield 6 Maps - Large & Small Maps Ranked"
[3]: https://kotaku.com/every-battlefield-6-map-ranked-from-worst-to-best-2000647370?utm_source=chatgpt.com "Every Battlefield 6 Map Ranked From Worst To Best"
[4]: https://www.ea.com/en/games/battlefield/battlefield-6/news/season-1-california-resistance-mid-season-roadmap?utm_source=chatgpt.com "Battlefield 6 and REDSEC Season 1"
[5]: https://battlefield.fandom.com/wiki/Blackwell_Fields?utm_source=chatgpt.com "Blackwell Fields | Battlefield Wiki - Fandom"
```

**Q: Which vehicles do you recommend in this conquest FF ON portal configuration?**

**A:** Recommended answer:
Run “combined-arms light” rather than full vehicle spam. Limit heavy armor and air, keep good transport coverage, and tune counts per map.

Below assumes your Conquest FF-ON rotation is:
Mirak Valley, Siege of Cairo, Liberation Peak, Iberian Offensive, Operation Firestorm, Eastwood.

---

## 1. Per-team baseline for big open maps

Use this layout on the fully combined-arms maps:

* Mirak Valley (everything: tanks, IFV, helis, jets)
* Liberation Peak (large, strong vehicle focus)
* Operation Firestorm (classic large vehicle map)
* Eastwood (infantry + light/heavy armor + helis + jets)

Per team, target:

* Main Battle Tanks (MBT): **2 active**
* Infantry Fighting Vehicles (IFV): **1**
* Mobile AA: **1**
* Attack Helicopter: **1**
* Jets: **1 total** (either 1 attack or 1 fighter; not both at once)
* Light Ground Transport / APC: **3–4 slots**

  * Prefer the new **Traverser Mark 2** (armored APC) if available in Portal; it’s much more survivable and popular than the base Light Ground Transport jeep, which the community widely considers a “death trap”.

Respawn / multipliers (if Portal exposes them):

* Heavy vehicles (MBT/IFV/AA): **0.75–1.0×** default spawn rate
* Air (heli/jet): **1.0–1.25×** default (slower is fine; they farm hard with FF ON)
* Light transports: **1.0–1.5×** default so infantry can actually move

Why:
You retain the “real Battlefield” feel (tanks + helis + jets) but cap each category to avoid a 100-player FF-ON lobby turning into nonstop splash/TK spam. MBT, attack heli, and jets are all top-tier power pieces in BF6; limiting their simultaneous presence per team keeps the match playable.

---

## 2. Urban / mixed maps with vehicles

### Siege of Cairo (urban vehicle map)

Siege of Cairo is explicitly designed as infantry vs tanks in a dense city layout.

Per team:

* MBT: **1–2** (start with 1 if FF grief becomes a problem)
* IFV: **1**
* Mobile AA: **0–1** (include 1 only if you run serious air)
* Attack Helicopter: **0–1** (1 max; consider disabling jets entirely here)
* Jets: **0** (not needed; air is already oppressive over narrow streets)
* Light transports / APC: **3–4**

This keeps armored “street fights” without turning main avenues into unplayable HE corridors once FF is on.

### Iberian Offensive (light-vehicle map)

Iberian Offensive is close-quarters infantry with only **Light Transports + IFVs** available by default.

Lean into that:

* MBT: **0**
* IFV: **1–2**
* Mobile AA: **0**
* Helis / Jets: **0**
* Light transports / APC: **3–4**

This makes Iberian your primary “tactical infantry + light armor” map in the rotation. With FF ON, it plays much cleaner than trying to force tanks/air into those tight Gibraltar streets.

---

## 3. Things to avoid or minimize with FF ON

1. Excessive LGT spam

   * Base **Light Ground Transport (LGT)** is widely criticized for being fragile and unpopular.
   * Don’t flood the map with them; use a moderate number and, if available, enable Traverser Mark 2 or similar APC transports instead.

2. Too many simultaneous MBTs and air

   * Mirak Valley’s default Conquest layout can feature up to three MBTs per side plus helis and jets.
   * In FF-ON public play, that much HE splash is a teamkill factory. Reduce tank count and air slots as above.

3. Vehicle-only or air-only presets on a FF-ON, blind-join 100p server

   * These are fun as niche modes, but they are rough on randoms and will spike rage-quits.

---

## 4. Practical Portal setup summary

If Portal lets you configure per-class counts:

* Global preset: **Combined Arms / All Vehicles**
* Then per map, per team:

  * Mirak / Lib Peak / Firestorm / Eastwood:
    `2 MBT, 1 IFV, 1 Mobile AA, 1 Attack Heli, 1 Jet, 3–4 LGT/APC`
  * Siege of Cairo:
    `1–2 MBT, 1 IFV, 0–1 Mobile AA, 0–1 Attack Heli, 0 Jets, 3–4 LGT/APC`
  * Iberian Offensive:
    `0 MBT, 1–2 IFV, 0 AA, 0 Air, 3–4 LGT/APC`

If Portal only exposes coarse toggles (e.g., “Tanks Only / Jets Only / Infantry Only / All Vehicles” as TheGamer’s guide suggests), then:

* Use **All Vehicles** for Mirak / Lib Peak / Firestorm / Eastwood.
* Use **Tanks + IFV only** (disable jets/helis if that’s an option) for Siege of Cairo.
* Use **IFV + light transports only** for Iberian Offensive.

If you tell me what exact vehicle sliders/toggles you see in your Portal host screen, I can translate this into a literal click-by-click configuration.

**Q: Is there a recommended faction or just the map default?**

**A:** Use the map’s default (era-appropriate) factions for your main Conquest rotation. Only override factions when you’re deliberately running a themed or asymmetric “gimmick” mode.

Reasoning is below.

---

### 1. How factions work in Portal

In Battlefield Portal (Battlefield 2042):

* You can choose factions from:

  * Battlefield 2042
  * Battlefield 3
  * Battlefield: Bad Company 2
  * Battlefield 1942
* For each experience you can select:

  * `Map Default` factions, or
  * A specific era’s factions (e.g., BF3 US vs BF3 RU, BF1942 UK vs Germany, etc.).

You can pit different era factions against each other, but you cannot mix two different factions on the same team.

---

### 2. Recommended choice for your FF-ON Conquest server

For a large public, FF-ON Conquest server:

1. Default / era-matching factions (recommended for main rotation)

   * On remastered classic maps (e.g., Caspian Border, Arica Harbor, Valparaiso, El Alamein, etc.), use the factions originally associated with that map (Map Default or that map’s era US vs RU/DE/UK).
   * On 2042 maps, use the standard 2042 factions.
   * Benefits:

     * Players instantly understand the “theme” and audio/visual cues.
     * Balance is generally tuned around those era loadouts.
     * Less confusion for randoms joining a blind public lobby.

2. Cross-era or asymmetric matchups (optional, for special events)

   * Example: BF1942 Germany vs BF2042 Specialists, or BF3 US vs 1942 Russia.
   * Only do this if you also:

     * Restrict modern weapons/vehicles so the match isn’t totally lopsided.
     * Advertise it clearly in the server name/description as a “theme” mode.
   * These are fun for one-off events, not ideal as your default 24/7 rotation.

3. Symmetry vs flavor

   * For lowest friction: same era on both sides (e.g., BF3 US vs BF3 RU, or pure 2042 vs 2042).
   * If you want flavor, use the original default (e.g., US vs Germany on 1942 maps, US vs RU on BC2/BF3 maps), but avoid anything that gives one side clearly superior equipment without compensation via rules.

---

### 3. Practical setup guideline

In Portal Builder or the in-game Portal config:

* For your “main” Conquest FF-ON 100p experience:

  * `Factions` → `Map Default` (or explicitly pick the matching era’s US/RU/UK/DE for each map).
* For any event or gimmick server:

  * `Factions` → choose specific cross-era pairing,
  * Then go into Weapons/Vehicles and cut back modern gear so both sides are playable.

So: use “map default” / era-correct factions for the core server, and only deviate when you’re intentionally running a themed or experimental mode and are willing to rebalance the gear around it.

**Q: Any recommendations on adjusting tickets per kill or reinforcement multiplier for this portal configuration?**

**A:** Best approach:
Leave “tickets per kill” at the default (1 ticket/death) and adjust round length using the *Reinforcement Multiplier* (and optionally a hard time limit). For your 100-player FF-ON Conquest server, start around a 1.25×–1.5× multiplier.

---

### 1. How the knobs actually work in Portal Conquest

* **Tickets per kill**

  * In Conquest, each death consumes 1 ticket by default; ticket bleed comes from flag control.
  * Portal does not expose a simple “tickets per kill = N” slider in the Core UI; to change it you’d use the Rules Editor and custom logic (not recommended for your first public config).

* **Reinforcement Multiplier (Core → Game Mode Details)**

  * This is the standard way to scale total tickets for Conquest.
  * Example: EA’s own answer shows `Reinforcement Multiplier = 1.5` giving ~1200 tickets vs the default.
  * Multiplying by 1.5 ≈ 50% longer round if player count and behavior stay similar.

* **Rules Editor: SetGameModeTargetScore**

  * In the Rules Editor you can set an explicit ticket target using `SetGameModeTargetScore` on `OnGameModeStarted`, instead of (or in addition to) the multiplier.

---

### 2. Practical recommendations for your 100-player FF-ON Conquest

Assumptions:

* 100 players, FF ON, “combined-arms light” vehicle setup as we outlined before.
* You want public, blind-join friendly matches that feel substantial but not marathon.

Recommended baseline:

* **Reinforcement Multiplier:** `1.25×`

  * This modestly extends rounds vs stock while 100 players and FF-ON accelerate ticket loss.
  * Expect ~20–30 minute rounds under normal pub behavior.

* **If you want longer “operations” style rounds:**

  * Try `1.5×`
  * Rounds trend toward 30–40 minutes. Above that, churn tends to spike; people rarely stay for 45+ minutes in random lobbies.

* **If you prefer shorter, high-turnover rounds:**

  * Use `1.0×` (default).
  * With 100 players and FF ON, rounds will probably be in the 15–25 minute band depending on how aggressively flags are played.

* **Time limit:**

  * Set a hard round time (e.g. 30 or 35 minutes).
  * This prevents the occasional stalemate from turning into a 50-minute slog even at high multipliers.

---

### 3. When (and when not) to touch tickets per kill

Only consider custom ticket-per-death logic via the Rules Editor if you want a *very* specific mode, for example:

* “No bleed on death, bleed only from flags”
* “Extra penalty for vehicle deaths”
* “Reduced penalty for squad revive meta”

Those require custom rules (e.g., tracking deaths and manually calling `SetGameModeTargetScore` / `SetGameModeScore`), and are easy to mis-tune for a blind-join 100-player server.

For a first public FF-ON Conquest Portal server:

* Keep the **default 1 ticket per death**.
* Use **Reinforcement Multiplier ≈ 1.25** and a **time limit** to shape match length.
* Adjust up to 1.5 or down to 1.0 after you see average round duration and player retention on your own server.

**Q: Do you recommend I reduce flag neutralization time?**

**A:** Short answer:
No, I do not recommend reducing flag neutralization time for your first 100-player FF-ON Conquest server. Keep neutralization (and capture) at default, or make only very small changes if you later see specific problems.

Details:

1. What faster neutralization does in 100-player FF-ON

   * Flags flip more often from small back-capping squads.
   * Frontlines become unstable and “whack-a-mole,” especially with 100 players.
   * Defenders get less time to react to a neutralizing flag, which is already harder with FF ON (you must be more careful shooting into smoke/doorways).
   * Combined with your increased ticket multiplier, rounds can end faster than intended.

2. Why default timing is usually best here

   * Default neutralization/cap times are tuned around large public Conquest: enough time to rotate, but not so long that stalemates are guaranteed.
   * With 100 players and FF ON, the effective pressure on flags is already higher than stock (more bodies, more accidental deaths).
   * You are already using reinforcement multiplier and possibly a time limit to control round length; using all three knobs at once (tickets, time, capture speed) makes behavior harder to predict.

3. When to consider a small reduction
   Only adjust after you’ve seen a few nights of gameplay and can identify a consistent issue:

   * If you see repeated, long stalemates on specific mid flags (e.g., both teams locked on a central sector) and very few flips, you could test a small global reduction:

     * Capture/neutralization rate scalar from `1.0 → 0.9` (≈10% faster), not more.
   * Watch average round duration and player churn; if rounds become too short or chaotic, revert to 1.0.

4. Practical recommendation for your config

   * Initial setup:

     * Neutralization time: default.
     * Capture time: default.
     * Reinforcement multiplier: ~1.25.
     * Optional time limit: 30–35 minutes.
   * Later tuning:

     * Touch neutralization only if you clearly see stalemates, and even then, change it slightly and test.

So for now: leave neutralization time at default. Let tickets and time limits do the pacing, and only revisit neutralization if real games show a consistent problem.

**Q: Enable AIM Assist Zoom Snap for this portal?**

**A:** Recommendation:
Do not enable Aim Assist Zoom Snap for this public 100-player FF-ON Portal server, unless you explicitly target controller-only or very casual mixed-input lobbies.

Use standard (non-snap) aim assist for controllers; keep snap disabled.

Reasoning:

1. Effect of Zoom Snap in large FF-ON lobbies

   * Zoom Snap strongly “locks” toward targets when you ADS (aim-down-sights).
   * In 100-player FF-ON chaos, this:

     * Increases accidental tracking onto teammates crossing your line of fire.
     * Feels unfair to mouse/keyboard players if cross-play is on.
     * Encourages low-effort “ADS spam” instead of precise aiming.

2. Input balance considerations

   * With cross-play on, Zoom Snap is one of the most complained-about assist settings; it’s seen as over-buffing controllers relative to M+K.
   * Standard rotational/friction aim assist is usually accepted; Zoom Snap is where debates start.
   * On a tactical FF-ON server, you want players thinking about trigger discipline, not relying on sticky snap.

3. When enabling Zoom Snap makes sense

   * Controller-only server (you explicitly restrict or discourage M+K).
   * Very casual, non-FF, short-TTK server where you prioritize accessibility over balance.
   * In those cases, advertise it clearly in the name/description.

4. Practical recommendation for your Portal config

   * Aim Assist: ON (so controller users aren’t punished).
   * Aim Assist Zoom Snap: OFF.
   * If you later decide to run a separate “Casual 100p” server, you can test Zoom Snap there, not on the FF-ON tactical instance.

So for this FF-ON Conquest portal: keep normal aim assist, disable Zoom Snap, and rely on your other tuning (vehicle limits, tickets, FF, and map choice) to shape the gameplay.

**Q: Enable Reload Entire Magazine for this portal?**

**A:** Recommendation:
Yes, you can enable “Reload Entire Magazine” for this FF-ON Conquest Portal server. It is generally positive for pacing and ammo management and does not conflict with your tactical goals.

Explanation and tradeoffs:

1. What “Reload Entire Magazine” does conceptually
   Different Battlefield titles implement reloads differently, but this toggle usually controls whether:

   * Partial mags are preserved and cycled (classic “magazines” system), or
   * A reload always refills the weapon to full and discards any leftover rounds (“replenish to full” with no partials).

   In practice this affects:

   * How punishing early or panic reloads are.
   * How important deliberate ammo management is.

2. Why enabling it is acceptable (and often preferable) here
   For a large 100-player FF-ON Conquest server:

   * High-intensity environment:
     With 100 players, firefights are frequent and you reload a lot. Tracking partial mags precisely is cognitively expensive and not obvious to most players.

   * Fewer “gotcha” moments:
     Without full-mag reload, players who reload at 20/30 repeatedly can end up with very few rounds overall, which feels like a trap in public play. Full-mag reload is more forgiving.

   * FF-ON balance:
     FF ON already increases the cost of sloppy fire. You do not need ammo micro-management to add more punishment. Let FF and your ticket system be the main discipline tools.

3. When you might disable it instead
   Consider disabling Reload Entire Magazine only if:

   * You are intentionally running a hardcore mil-sim style mode with:

     * Higher damage, reduced HUD, FF ON, and
     * A target audience that explicitly wants realistic magazine handling.
   * In that case, advertise it clearly: “mag realism, no full-mag reloads.”

4. Practical recommendation for your portal:

   For your current “tactical but public-friendly” FF-ON Conquest:

   * Reload Entire Magazine: ON
   * Combined with:

     * FF ON, default damage (or slight increase),
     * Limited vehicles,
     * Default capture/neutralization times,
     * Reinforcement multiplier ~1.25 and 30–35 min time limit.

This setup keeps the shooting model readable and fair for randoms, while your other settings carry the tactical and discipline aspect.

**Q: Disable Third Person View on vehicles in this portal?**

**A:** No, I do not recommend disabling third-person view for vehicles on your main 100-player FF-ON Conquest Portal server.

Reasoning:

1. Situational awareness and FF ON

   * Third-person greatly improves awareness of infantry around the vehicle, obstacles, and angles.
   * With FF ON, better visibility means fewer accidental roadkills and fewer blind HE shots into friendlies.
   * First-person-only vehicles make it much easier to run over teammates or misjudge blast radius in tight spaces.

2. Accessibility and server popularity

   * Driving in pure first-person is harder and less comfortable for many players (FOV, motion, cockpit obstruction).
   * On a blind-join public server, removing third-person will push a lot of casual players away from vehicles entirely.
   * That usually leads to:

     * Underused vehicle slots.
     * A small minority of “vehicle specialists” dominating, or vehicles sitting empty.

3. Tactical vs “hardcore mil-sim” identity

   * You are already FF ON + tuned vehicles + tuned tickets; that’s enough to make the server feel tactical.
   * First-person-only vehicles is a good lever for a separate, explicitly “Hardcore” server, not the main public instance.

4. Recommended approach for your config

   * Main FF-ON Conquest server:

     * Vehicle camera: allow both first-person and third-person.
     * Let players pick per situation; third-person helps avoid FF incidents.
   * Optional second “Hardcore” profile:

     * FF ON, higher damage, reduced HUD, and vehicle third-person disabled.
     * Make that identity explicit in the server name.

So for the server you are building now: keep third-person view enabled for vehicles. It supports FF discipline, keeps vehicles usable for most players, and protects the server’s popularity.

**Q: If enabled, then I can enable all vehicles in all maps without micromanaging!**

**A:** Not quite. Third-person view makes vehicles safer and more usable, but it does not remove the need for at least minimal per-map tuning if you want the server to play well with FF ON.

Key points:

1. What third-person actually solves

* Better awareness → fewer accidental roadkills and fewer blind HE shots into teammates.
* More players are willing to drive/fly without feeling “blind.”
* This lets you safely allow *more* vehicles than you otherwise would.

2. What it does *not* solve

* Tight maps (Iberian-style streets, dense urban flags) will still get overwhelmed if you throw full jets, attack helis, MBTs, IFVs, LAVs, etc. at them simultaneously.
* On a 100-player FF-ON server, heavy vehicles with splash in tight lanes will still spike TKs and frustration even with third-person enabled.

3. If you truly want “no micromanagement”
   Fast, low-effort setup that’s still playable:

* Global vehicle preset: **All Vehicles**.
* Third-person: **Enabled** for all vehicles.
* Reinforcement multiplier: ~**1.25×**; normal damage; FF ON.

Accept that:

* Some maps (tightest urban) will be loud/chaotic.
* TK rate will be higher than ideal.
  If you’re okay with a “mayhem” identity, this is fine.

4. Minimal micro that gives a big payoff
   If you’re willing to do *just a little* map-level adjustment:

* Large/open maps (Mirak / Lib-Peak / Firestorm / big S1 map):

  * **All vehicles** allowed.
* Dense urban / tight maps (Cairo, Iberian-style):

  * **No jets**, at most **1 attack heli**, and **0–1 MBT per team**.
  * Keep light transports + IFVs.

That’s only 1–2 tweaks per map and dramatically improves FF-ON play, while third-person keeps vehicles usable.

Summary:
Enabling third-person makes it *safer* to run generous vehicle settings, but if you want a “good” tactical server rather than pure chaos, you should still do at least a small amount of per-map vehicle tuning.

**Q: Enable vehicle health regeneration for this portal?**

**A:** Yes. Enable vehicle health regeneration for this portal, but keep it conservative rather than aggressive.

Recommended baseline for your 100-player FF-ON Conquest server:

* Vehicle Health Regeneration: **Enabled**
* Regeneration Delay: **slightly longer than default**

  * Example: if default is ~10 s, use **12–15 s** after last damage.
* Regeneration Rate: **near default or a bit slower**

  * Example: **0.75–1.0×** default rate.

Reasoning:

* FF ON already punishes sloppy vehicle play (accidental TKs, careful HE use).
* With limited vehicles per team, you don’t want them permanently crippled or forced to retreat to engineers every time they take chip damage.
* A slightly slower regen with a longer delay:

  * Keeps vehicles viable over a full round.
  * Prevents “unkillable” armor that ducks behind cover for 3 seconds and comes back full.
  * Reduces maintenance micromanagement for random public players.

Only consider fully disabling regen on a separate, explicitly “Hardcore / Mil-Sim” server, not on your main public FF-ON Conquest instance.

**Q: which experience image do you recommend for this portal?**

**A:** Use a modern, combined-arms Conquest image that matches your main map pool and “tactical FF ON” identity.

Concrete recommendation:

1. Primary choice

   * Pick the **Mirak Valley / large open combined-arms Conquest** key art (or the closest equivalent in your Portal image list: modern soldiers + tank/IFV in a wide outdoor environment).
   * Reason: your server identity is “100p tactical Conquest with vehicles, FF ON, no bots.” Mirak-style art instantly signals:

     * Modern era (not WW2 / 1942).
     * Mixed infantry + armor, not BR or infantry-only.
     * Large-scale outdoor fighting → fits 100 players.

2. If you want to emphasize urban play instead

   * Use the **Siege of Cairo / dense modern city Conquest** art (soldiers in streets, city skyline), but still clearly Conquest (not Redsec/BR branding).
   * Good if you think Cairo will be your “signature” map.

3. If the picker only shows generic icons

   * Choose the **Conquest-style tile** that shows:

     * A soldier + vehicle (tank/APC),
     * Not a parachuting/BR silhouette, not a WW2 helmet.
   * Avoid:

     * Battle royale / Redsec-branded images.
     * WW2 or BC2-era art, unless your rotation is actually built around those eras.

Rule of thumb:

* Modern combined-arms Conquest art that visually matches “100-player, FF ON, tactical but public-friendly” will attract the right players and avoid people expecting BR, infantry-only, or WW2 gimmicks.

**Q: What Welcome Message and Periodic Message do you recommend on this portal?**

**A:** Recommended text:

**Welcome Message (on join)**

> Welcome to NA-E 100p Tactical Conquest | FF ON. Play the objective, check your fire (FF ON, vehicles + explosives), no griefing or spawn camping. Repeated teamkilling, hate speech, or exploiting = kick/ban.

**Periodic Message (every few minutes)**

> Reminder: FF ON – watch fire around flags and vehicles. Play the objective, keep chat clean, no intentional TK or griefing. Report problem players in chat so an admin can review.

**Q: Oh wow, lobbies populate slower, without bots!**

**A:** Yes, that’s expected. With no bots, your server has to “seed” with real players before it snowballs.

You have three main options:

1. Accept slower start, keep high-quality games

* Keep bots OFF, FF ON, tactical settings.
* Best for quality and future progression if DICE tightens XP rules again.
* Strategy:

  * Play with a small squad at predictable times (e.g., NA-E evenings).
  * Keep rotation short (3–4 maps) so people learn what to expect.
  * Use a consistent, searchable name so returning players can find it.

2. Use bots only for seeding

* Turn ON limited bot backfill (e.g., up to 20–30 bots) with Normal+ difficulty.
* Once you hit a decent human count (50–60+), manually switch to a bot-free version of the same config (a second “No Bots” experience) for peak hours.
* Downside: extra admin work, and XP may be partially restricted while bots are present.

3. Go “mayhem” to fill faster, then tighten later

* Temporarily loosen rules:

  * FF OFF, all vehicles, bots allowed, aim assist more generous.
* Use that config to attract population and regulars.
* Later spin up your “serious” FF-ON, no-bot tactical server and advertise it in the name/description of the casual one.

If your goal is genuinely tactical FF-ON play, I’d do:

* Day 1–3: allow a modest number of bots (so lobbies feel alive), clearly labeled:
  `NA-E | 100p Tactical Conquest | FF ON | Bots Fill to X`
* Once you see recurring players / Discord interest: bring up a second, pure-human server and encourage migration there during peak hours.

**Q: I would like both backfill with lower difficulty on a twin portal!**

**A:** Set up two nearly identical experiences and only change the AI section on the “seed” version.

### 1. Twin-portal layout

Main server (no bots, humans only)

* Name:
  `NA-E | 100p Tactical Conquest | FF ON | No Bots`
* Core settings:

  * Conquest, 100 players
  * FF ON
  * Vehicle and ticket settings as already defined
  * AI soldiers: OFF

Seed server (bot backfill, lower difficulty)

* Name:
  `NA-E | 100p Tactical Conquest | FF ON | Bots Fill to 40`
* Purpose: keep the server feeling alive while it’s building population.

Both should:

* Use the same map rotation.
* Use the same flag, vehicle, damage, HUD, and ticket settings.
* Be persistent servers in the same region (NA-E).

---

### 2. How to create the twin portals

1. In Portal Builder:

   * Open your current FF-ON Conquest experience.
   * Use “Duplicate / Copy” (or re-create with identical settings).
   * Rename the copy to the “Bots Fill to 40” version.

2. In the duplicated “seed” experience, change only the AI section:

   AI / Bots settings (seed server)

   * AI Soldiers: ENABLED
   * AI Backfill: ENABLED
   * Max AI Soldiers: 30–40

     * Example: “Fill server to 40 players with AI”; leaves 60+ slots for humans.
   * Bot Difficulty: EASY or one step below default

     * Enough to be killable and not oppressive; this is a warmup/seeding environment.
   * Team Balancing: ON
   * AI Vehicle Usage (if exposed):

     * Allow transports and light armor.
     * Optionally disable attack helis/jets for bots to reduce chaos.

3. Keep FF ON on both servers for identity consistency, or:

   * If you want the seed server less punishing, set:

     * Seed server: FF OFF (casual fill).
     * Main server: FF ON (tactical).
   * That is optional; your original request works fine with FF ON on both, bots just on lower difficulty.

---

### 3. Recommended welcome/periodic messages for the seed server

Welcome (seed server)

> Tactical Conquest with FF ON and AI backfill up to 40 players. Play the objective, check fire around AI and teammates, no griefing. When the human player count is high, please move to the “No Bots” twin server in the browser.

Periodic (seed server)

> FF ON, bots fill to 40 to seed lobbies. For full human matches, look for: “NA-E | 100p Tactical Conquest | FF ON | No Bots” in the browser.

---

### 4. Operational pattern

* Keep both servers persistent.
* Play and advertise the seed server early; once human count is ~60+, switch yourself and announce in chat to move to the No Bots server.
* Over time, regulars will learn the pattern and search for the No Bots instance first during peak hours.
