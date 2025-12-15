## OpenAI ChatGPT v1.2025.336 Detecting.Frame.Rate.md

**Q: In Unreal Engine, how do I detect the frame rate?**

**A:** Use `Get World Delta Seconds` (Blueprint) or `UWorld::GetDeltaSeconds()` (C++) and compute `FPS = 1 / DeltaSeconds`.

Blueprint options

* Instantaneous FPS (per-frame):

  * `Event Tick` → `Get World Delta Seconds` → `1.0 / DeltaSeconds`
  * Guard against tiny values: `Max(DeltaSeconds, 0.0001)`
* Smoothed FPS:

  * `Event Tick` → `Get World Delta Seconds`
  * `Interp To` or running average over N samples, then `1 / SmoothedDeltaSeconds`
* Built-in stats (debug only, not for gameplay logic):

  * Console: `stat fps` (shows FPS and frame time)

C++ options

* Instantaneous:

  * `const float DeltaSeconds = GetWorld()->GetDeltaSeconds();`
  * `const float FPS = (DeltaSeconds > SMALL_NUMBER) ? (1.0f / DeltaSeconds) : 0.0f;`
* Smoothed engine value (if enabled/available in your context):

  * `GAverageFPS` / `GAverageMS` are engine globals often used for stats displays (not a stable API contract across versions).

Notes

* For gameplay decisions, prefer delta time (time-based movement) rather than branching on FPS.
* For UI display, use a smoothed FPS to avoid flicker.
