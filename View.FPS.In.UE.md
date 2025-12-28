## OpenAI ChatGPT v1.2025.350 View.FPS.In.UE.md

**Q: How do I view (in editor) FPS in unreal engine?**

**A:** In-editor FPS display options:

* Viewport overlay (quick)

  * In the viewport: Show → Stats → enable FPS (and optionally Unit / UnitGraph).

* Console command (PIE or editor viewport)

  * Open console with `~` and run:

    * `stat fps` (FPS + frame time)
    * `stat unit` (Game / Draw / GPU frame times)

* Editor setting (persistent)

  * Edit → Editor Preferences → search “FPS” → enable “Show Frame Rate and Memory” (shows FPS + memory in the editor UI).

**Q: Can I show in game?**

**A:** Yes. In-game (PIE or packaged) options:

* Console (runtime)

  * Press `~` and run:

    * `stat fps`
    * `stat unit`
  * Works in PIE and packaged builds if console is enabled.

* Blueprint (toggleable HUD text)

  * Enable **Project Settings → Engine → General Settings → Framerate → Smooth Frame Rate** as needed.
  * Use `Get World Delta Seconds` → FPS = `1 / DeltaSeconds`.
  * Display via UMG TextBlock or `Draw Text` on HUD.
  * Optional: gate behind a debug key.

* C++ (engine stats overlay)

  * Call `GEngine->Exec(GetWorld(), TEXT("stat fps"));`
  * Or expose a console command and bind to input.

Notes:

* `stat fps`/`stat unit` are development tools; typically disabled or hidden in Shipping unless explicitly allowed.
* For Shipping builds, implement the UMG/Blueprint method if you need persistent player-visible FPS.
