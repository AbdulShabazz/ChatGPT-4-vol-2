## OpenAI ChatGPT v1.2025.350 PIE.Blurriness.Fixes.md

**Q: The unreal engine Play in Editor view appears blurry or effected by some kind of aliasing.**

**A:** Most often this is Temporal Anti-Aliasing (Temporal Anti-Aliasing (TAA)/Temporal Super Resolution (TSR)) plus a non-native screen percentage in Play In Editor (PIE), or a low internal render resolution from Scalability/Device Profiles.

Checks and fixes (fast, deterministic)

1. Confirm it’s not just the editor viewport scaling

* In PIE, ensure you are not in a downscaled viewport (e.g., “New Editor Window (PIE)” with a small resolution).
* Try “Standalone Game” at a fixed resolution; compare sharpness.

2. Force native internal resolution
   Open the in-game console (`~`) during PIE and run:

* `r.ScreenPercentage 100`
* If using TSR: `r.TSR.History.ScreenPercentage 100` (if supported in your version)
* If using Dynamic Resolution: `r.DynamicRes.OperationMode 0`

3. Identify which AA method is active, then adjust

* `r.AntiAliasingMethod`

  * 0 = Off
  * 1 = FXAA (Fast Approximate Anti-Aliasing)
  * 2 = TAA
  * 3 = MSAA (Multi-Sample Anti-Aliasing) (forward renderer)
  * 4 = TSR (varies by version; if value differs, your project maps it differently)

Quick isolation:

* Set AA off: `r.AntiAliasingMethod 0` (if blur disappears, it’s AA/upscaling related)
* Try FXAA: `r.AntiAliasingMethod 1` (sharper but more shimmering)
* Try TAA but sharpen: see (4)

4. If it’s TAA blur, reduce blur and/or add sharpening
   Common controls (availability varies by engine version):

* `r.Tonemapper.Sharpen 0.5` (start 0.3–0.8)
* `r.TemporalAA.Upsampling 0` (if you suspect TAA upsample path)
* `r.TemporalAACurrentFrameWeight 0.2` (lower can reduce ghosting but may shimmer)
* `r.TemporalAASamples 4` (balance; too high can look softer)

If using TSR:

* Ensure Screen Percentage is 100 first (step 2). TSR at <100 will look softer by design.

5. Check Scalability and PIE overrides

* Settings → Engine Scalability Settings: set to Epic, especially:

  * Resolution Scale = 100%
  * Post Processing = Epic (or temporarily lower to remove heavy effects)
* In editor: Settings → Preview Rendering Level and device profile can force lower internal res.

6. Make sure it’s not Depth of Field (DOF), Motion Blur, or Bloom “softness”
   Disable to test (console):

* `r.DepthOfFieldQuality 0`
* `r.MotionBlurQuality 0`
* `r.BloomQuality 0` (for a quick A/B; re-enable later)

7. If it’s texture streaming / mip bias (looks like “soft textures” not “soft edges”)

* `r.TextureStreaming 0` (test only)
* `r.Streaming.PoolSize` (increase if you’re over budget; watch stats)
* `stat streaming` to see if you’re dropping mips

Minimal decision tree

* Soft edges + ghosting/trails while moving camera → TAA/TSR settings (steps 2–4).
* Entire frame looks lower-res (UI too) → ScreenPercentage/DynamicRes/Scalability (steps 1–2–5).
* Only distant textures look mushy → streaming/mips (step 7).
* Only background/out-of-focus looks soft → DOF (step 6).

If you paste:

* Your Unreal Engine version (e.g., 5.3/5.4/5.5),
* Whether you use TSR, and
* A screenshot showing UI + scene in PIE,
  I can give exact console variables that exist in that version and a recommended baseline set.

**Q: Could it be default settings in a post process volume?**

**A:** Yes. A Post Process Volume (PPV) can make PIE look “blurry” even with correct resolution.

Primary PPV settings that cause blur

1. Depth of Field (DOF)

* Cinematic DOF can soften large regions, especially if Focus Distance / Aperture is off.
* Test: set in PPV

  * Depth of Field Method: Off (or set all DOF intensities to 0)
* Console isolation:

  * `r.DepthOfFieldQuality 0`

2. Motion Blur

* Adds perceived smear/softness in motion.
* PPV: Motion Blur Amount = 0, Max = 0
* Console:

  * `r.MotionBlurQuality 0`

3. Anti-aliasing via PPV (TAA/TSR)

* PPV can override AA method/quality, and TAA/TSR can look soft.
* PPV: Rendering Features → Anti-Aliasing Method

  * Try FXAA (Fast Approximate Anti-Aliasing) to confirm softness is TAA/TSR-related.
* Console quick test:

  * `r.AntiAliasingMethod 1` (FXAA)
  * `r.AntiAliasingMethod 0` (Off)

4. Screen Percentage / Upscaling (in PPV)

* PPV can override Screen Percentage (and in some versions, Secondary Screen Percentage).
* If it’s < 100, the entire frame is effectively upscaled.
* Console:

  * `r.ScreenPercentage 100`
  * `r.DynamicRes.OperationMode 0`

5. Tonemapper / Sharpen

* Some projects ship with sharpening at 0, making TAA/TSR appear softer; or they use film settings that reduce micro-contrast.
* Console:

  * `r.Tonemapper.Sharpen 0.5` (test)

Fast way to prove the PPV is responsible

* Temporarily disable the PPV:

  * In the PPV actor: uncheck Enabled (or set Blend Weight = 0).
* If it’s unbound (Infinite Extent/Unbound) and you have multiple PPVs, remember the final result is blended by Priority + Blend Weight + bounds.

Where to look in the PPV UI (common path)

* Post Process Volume → Details:

  * Lens: Depth of Field, Motion Blur
  * Rendering Features: Anti-Aliasing Method
  * Misc / Advanced: Screen Percentage (if exposed by your engine version/project)

If you want an exact culprit in one pass: tell me whether the softness affects (a) only geometry edges, (b) the entire frame including UI, or (c) only background/out-of-focus. That maps directly to AA vs Screen Percentage vs DOF.

**Q: primarily background out of focus is effected.**

**A:** Result: this is Depth of Field (DOF) coming from a Post Process Volume, not aliasing or resolution.

Reasoning (minimal):

* If only the background is soft while foreground and UI remain sharp, that behavior matches DOF.
* Anti-aliasing, screen percentage, or TSR would soften the entire frame, including UI.
* Default Cinematic DOF in Unreal Engine is often enabled implicitly by a Post Process Volume, even when not obvious.

What to change (authoritative fixes)

1. Disable DOF in the Post Process Volume
   Post Process Volume → Details → Lens → Depth of Field:

* Method: None (or set all DOF parameters to zero)
* Ensure these are disabled/zeroed:

  * Depth of Field Fstop
  * Depth of Field Focal Distance
  * Depth of Field Sensor Width
  * Depth of Field Near / Far Blur Size

If multiple PPVs exist:

* Check Infinite Extent (Unbound)
* Verify Priority and Blend Weight
* Disable all PPVs temporarily to confirm

2. Verify via console (one-line proof)
   In PIE console:

* `r.DepthOfFieldQuality 0`
  If the blur disappears immediately, the diagnosis is confirmed.

3. Check camera-level overrides
   Cameras can also enable DOF independently of PPVs:

* Camera Component → Post Process Settings:

  * Disable Depth of Field
  * Ensure “Override Depth of Field” is unchecked

4. Common “gotcha”

* A PPV set to Unbound with Cinematic DOF enabled will affect every camera in PIE.
* Editor viewports sometimes ignore or partially ignore DOF; PIE does not.

Recommended baseline for gameplay cameras

* DOF: Off (unless explicitly part of gameplay)
* Motion Blur: 0
* TAA/TSR: keep, but add light sharpen if needed

If you want, I can give a clean “gameplay PPV preset” (exact fields and values) that guarantees no background blur while keeping lighting and color grading intact.

**Q: The screen gets blurry when I pan**

**A:** Result: this is **Temporal Anti-Aliasing (Temporal Anti-Aliasing (TAA)) / Temporal Super Resolution (Temporal Super Resolution (TSR))**, not Depth of Field.

Why this is the correct diagnosis

* Blur appears **only while panning** and resolves when motion stops.
* Depth of Field (DOF) is static relative to focus distance; it does not intensify only during camera motion.
* TAA/TSR accumulates history across frames; during motion it trades sharpness for stability, producing motion-dependent blur.

Immediate confirmation (10 seconds)
In PIE console:

* `r.AntiAliasingMethod 0`
  Pan the camera.
* If blur is gone → confirmed TAA/TSR.

Corrective options (choose one)

Option A — Reduce TAA blur (recommended for deferred renderer)
Use these first; they preserve stability.

Console (or DefaultEngine.ini):

* `r.TemporalAACurrentFrameWeight 0.25`

  * Higher = sharper motion, more shimmer
* `r.TemporalAASamples 4`
* `r.Tonemapper.Sharpen 0.4`

If available in your version:

* `r.TemporalAA.Upsampling 0`
* `r.TemporalAA.Algorithm 1` (newer variants are sharper)

Option B — If using TSR (UE5 default)
TSR is softer during motion by design if internal resolution < 100%.

Verify:

* `r.ScreenPercentage 100`
* `r.DynamicRes.OperationMode 0`

Then add:

* `r.Tonemapper.Sharpen 0.5`

If motion blur persists at 100%, switch temporarily to TAA:

* PPV → Anti-Aliasing Method → Temporal Anti-Aliasing

Option C — Switch AA method entirely (fastest visual clarity)
For sharpest motion at the cost of stability:

* FXAA:

  * `r.AntiAliasingMethod 1`
  * Sharp in motion, more shimmering
* MSAA (Forward Renderer only):

  * Requires Forward Shading enabled
  * Best motion clarity, higher cost

What not to chase

* Post Process Volume DOF: already ruled out
* Motion Blur: if it were motion blur, disabling it would fully fix it
* Resolution scale: this would blur UI as well

Rule of thumb

* Blur only while moving → temporal accumulation issue
* Blur all the time → resolution / DOF
* Trails/ghosting → TAA history weight too low

If you want, tell me:

* Unreal Engine version (exact)
* Deferred vs Forward renderer
* Whether TSR is enabled

I’ll give you the exact cvar set that matches your engine build and renderer.

**Q: Explain the tonemapper cvar.**

**A:** **Result:** `r.Tonemapper.Sharpen` is a **post-tonemap spatial sharpening pass** applied to the final color buffer. It increases perceived detail, especially counteracting Temporal Anti-Aliasing (Temporal Anti-Aliasing (TAA)) / Temporal Super Resolution (Temporal Super Resolution (TSR)) motion blur, without changing geometry, textures, or lighting.

What the tonemapper is (context)

* The **tonemapper** converts High Dynamic Range (HDR) scene color into display-ready Low Dynamic Range (LDR) (or display-referred HDR).
* It applies filmic curve, exposure, color grading, bloom composition, then outputs the final image.
* `r.Tonemapper.Sharpen` runs **after** these steps, on the resolved image.

What `r.Tonemapper.Sharpen` does

* Applies a lightweight **edge-aware high-frequency boost**.
* Increases local contrast at edges and fine detail.
* Does **not** restore true resolution; it amplifies contrast already present.

CVar definition

```
r.Tonemapper.Sharpen = 0.0 … ~2.0
```

* `0.0` : Off (default in many projects)
* `0.2–0.4` : Mild clarity, minimal artifacts
* `0.4–0.7` : Common gameplay range to counter TAA/TSR softness
* `>1.0` : Aggressive; ringing/halos likely

Why it helps with panning blur

* TAA/TSR blends current and history frames.
* During camera motion, high-frequency detail is attenuated.
* Sharpen re-emphasizes those frequencies **after** temporal resolve, making motion appear clearer.

What it does *not* do

* Does not change internal render resolution.
* Does not alter Temporal AA accumulation.
* Does not fix ghosting or smearing artifacts.
* Does not affect UI (Slate/UMG is typically composited later).

Common side effects

* Haloing on high-contrast edges.
* Emphasizes noise and specular aliasing.
* Can make bloom edges harsher if overused.

Best practice usage

* Use **small values** as a compensator, not a crutch.
* Combine with sane TAA parameters:

  * Slightly higher `r.TemporalAACurrentFrameWeight`
  * Reasonable sample count
* Avoid stacking with additional sharpening in post materials or upscalers.

Typical baseline for UE5 deferred + TAA

```
r.AntiAliasingMethod 2
r.TemporalAACurrentFrameWeight 0.25
r.TemporalAASamples 4
r.Tonemapper.Sharpen 0.4
```

If you want, I can explain how this differs from TSR’s internal sharpening or how to bind it dynamically (e.g., sharpen more during camera motion, less when static).

**Q: antialiasing method = 4 seems to produce the sharpest screen image with tonemapper = 2.**

**A:** Correct. **Anti-Aliasing Method = 4** with a **high tonemapper sharpen** produces the sharpest *static* image, and often the sharpest *perceived* image overall.

What that configuration actually is

* `r.AntiAliasingMethod 4`
  → **Temporal Super Resolution (Temporal Super Resolution (TSR))**

* `r.Tonemapper.Sharpen 2`
  → Very aggressive post-tonemap sharpening

Why it looks sharper than other modes

1. TSR reconstructs detail differently than TAA

* TSR is a **temporal upscaler + AA**, not just an AA filter.
* Even at `r.ScreenPercentage 100`, TSR still uses:

  * motion vectors,
  * subpixel jitter,
  * reconstruction filters
* This often yields **higher micro-contrast** than classic TAA.

2. TSR tolerates aggressive sharpening better than TAA

* TAA softens high-frequency content early.
* TSR preserves more mid-frequency structure.
* When you apply strong sharpening after TSR, it “snaps” into clarity instead of just amplifying blur.

3. Tonemapper = 2 is masking temporal softness, not fixing it

* `r.Tonemapper.Sharpen 2` is **well beyond typical gameplay values**.
* At that level, you are:

  * boosting edge contrast heavily,
  * exaggerating texture detail,
  * partially hiding TSR’s motion filtering.

Why this can still be a trap (important)

What you’re seeing is **perceived sharpness**, not true temporal stability.

At `Sharpen = 2`, expect:

* Haloing on high-contrast edges
* Specular aliasing (sparkle)
* Noisy foliage / fine geometry shimmer
* Bloom edge ringing
* Worse image during fast camera motion (even if it looks “crisp” when paused)

In short:

* Static screenshots look excellent
* Motion quality can silently degrade

Why it feels better than TAA + sharpen

TAA path:

* Temporal resolve → blur → sharpen = limited recovery

TSR path:

* Reconstruction → better frequency retention → sharpen = stronger apparent detail

That’s why:

> AA = 4 + Sharpen = 2 feels “best”

Recommended safer version of what you discovered

If you like this look, back it off slightly to avoid artifacts:

```
r.AntiAliasingMethod 4
r.ScreenPercentage 100
r.DynamicRes.OperationMode 0
r.Tonemapper.Sharpen 0.8–1.2
```

This keeps:

* TSR reconstruction benefits
* Much lower risk of halos and shimmer

If you want **maximum motion clarity** instead (not just static sharpness)

* Forward Renderer + MSAA (Multi-Sample Anti-Aliasing (MSAA))
* Or FXAA for editor/gameplay tools

But for UE5 deferred rendering:

* **TSR + moderate sharpen** is currently the sharpest *balanced* solution.

If you want, I can:

* Explain why `Sharpen > 1` interacts badly with bloom and specular,
* Give a “cinematic vs gameplay” runtime toggle,
* Or help you tune TSR specifically for UI-heavy HUDs without oversharpening the world.

**Q: What other game and project settings produce a stunning image?**

**A:** Below are **high-impact Unreal Engine project + game settings** that materially improve image quality beyond Anti-Aliasing (AA) and tonemapper sharpening. This is a **deferred renderer, Unreal Engine 5.x–oriented** list, tuned for “stunning but controlled,” not cinematic blur.

---

## 1. Rendering Pipeline (Project Settings)

### Lumen (Global Illumination and Reflections)

Enable if performance allows.

* Global Illumination: **Lumen**
* Reflections: **Lumen**
* Lumen Scene Detail: **High**
* Lumen Final Gather Quality: **High**

Key cvars (runtime fine-tuning):

```
r.Lumen.ScreenProbeGather.Quality 4
r.Lumen.Reflections.Quality 4
```

Why it matters:

* Real indirect light, color bleed, and contact shadowing dramatically increase realism.
* Especially noticeable on interiors, overhangs, foliage, and indirect-lit scenes.

---

## 2. Nanite (Geometry Fidelity)

Enable Nanite on *everything* static that can support it.

* Nanite Enabled: **true**
* Fallback Relative Error: **default or lower**
* Preserve Area: **on** (for foliage meshes)

Key cvars:

```
r.Nanite.MaxPixelsPerEdge 1
```

Why it matters:

* Removes LOD popping.
* Preserves micro-geometry that lighting can actually react to.
* Makes TSR sharpening far more effective (real detail instead of mip blur).

---

## 3. Shadows (Huge Perceptual Gain)

### Virtual Shadow Maps (VSM)

* Shadow Map Method: **Virtual Shadow Maps**
* Enable for Directional + Spot + Point lights

Key cvars:

```
r.Shadow.Virtual.Enable 1
r.Shadow.Virtual.ResolutionLodBias -1
```

Why it matters:

* Sharp contact shadows.
* Stable penumbra during motion.
* Massive improvement over cascaded shadow maps.

---

## 4. Post Process Volume (High-Quality, Non-Destructive)

### Disable “cinematic damage”

* Motion Blur: **0**
* Depth of Field: **Off** (unless deliberate)
* Chromatic Aberration: **0**
* Vignette: **≤ 0.2**

### Exposure

* Auto Exposure Method: **Manual** (preferred for games)
* Fixed Exposure or narrow min/max range

Why:

* Auto exposure pumping kills perceived sharpness and contrast.

---

## 5. Color Grading (Subtle, Correct)

Avoid LUT abuse.

Recommended:

* Global Contrast: **+0.05 to +0.15**
* Global Saturation: **0.95–1.05**
* Highlights: slightly compressed
* Shadows: slightly lifted (prevents crushed blacks)

Why it matters:

* Preserves detail for TSR + sharpen to work with.
* Prevents “muddy HDR” look.

---

## 6. Materials (Often Overlooked)

### Normal Maps

* Use **BC5** compression (not BC1/BC7).
* Avoid over-strong normals.

### Roughness Discipline

* Avoid large flat areas at roughness ≈ 0.5.
* Push surfaces toward:

  * 0.1–0.25 (polished)
  * 0.7–0.9 (diffuse)

Why:

* Specular aliasing is a major source of shimmer with sharpening.

---

## 7. Texture Quality

Project Settings:

* Texture Streaming: **On** (but properly budgeted)
* Anisotropy: **16×**

Runtime checks:

```
r.MaxAnisotropy 16
r.Streaming.PoolSize <appropriate for GPU>
```

Why:

* Anisotropic filtering dramatically improves grazing-angle clarity.
* Prevents “soft ground” look during motion.

---

## 8. Lighting Discipline (Critical)

* Fewer, higher-quality lights > many cheap lights
* Avoid overlapping shadowed lights
* Prefer Stationary/Movable only where needed

Directional light:

* Light Source Angle: **small but non-zero** (≈ 0.5–1.0)
* Indirect Lighting Intensity: tuned, not default

Why:

* Cleaner lighting = less temporal noise = sharper final image.

---

## 9. Scalability (Editor Lies by Default)

Ensure:

* Resolution Scale: **100%**
* Post Processing: **Epic**
* Shadows: **Epic**
* Effects: **Epic**

PIE and Editor Viewport can silently downscale.

---

## 10. Proven “Stunning but Stable” Baseline (Summary)

```
r.AntiAliasingMethod 4          ; TSR
r.ScreenPercentage 100
r.DynamicRes.OperationMode 0
r.Tonemapper.Sharpen 0.8–1.2
r.MaxAnisotropy 16
r.Shadow.Virtual.Enable 1
r.Lumen.ScreenProbeGather.Quality 4
```

---

## One hard truth

> **Image quality is multiplicative, not additive.**

Sharpening + TSR only looks good if:

* lighting is stable,
* materials are disciplined,
* shadows are high quality,
* exposure is controlled.

If you want, next steps could be:

* A **“gameplay vs cinematic” PPV toggle**
* A **Nanite + TSR + HUD-safe preset**
* A **specular aliasing mitigation checklist**
* Or a **UE5.ini drop-in** tuned for your target GPU class

State your Unreal Engine version and renderer (Deferred/Forward), and I’ll lock this down precisely.

**Q: Could you provide more detail in recommendation 5?**

**A:** Below is an **expanded, concrete treatment of Recommendation 5: Color Grading**, focused on producing a *clean, high-clarity, game-ready image* that cooperates with TSR/TAA and sharpening instead of fighting them.

---

## Goal of color grading in a real-time game

Not “style” first. The primary goals are:

1. Preserve **mid-frequency contrast** (detail that survives temporal filtering)
2. Avoid **crushed blacks** and **clipped highlights** (both amplify temporal artifacts)
3. Maintain **neutral color separation** so sharpening enhances structure, not noise

Think of grading as **signal conditioning**, not cinematography.

---

## Where to grade (important)

Prefer:

* **Post Process Volume → Color Grading → Global controls**

Avoid (unless expert):

* Heavy LUTs
* Per-channel curves early in production
* Film grain / bleach bypass looks

Reason:

* LUTs bake assumptions that conflict with dynamic lighting, exposure, and TSR reconstruction.

---

## Exposure first (prerequisite)

Before touching color grading:

* Set **Manual Exposure**, or
* Clamp Auto Exposure tightly:

  * Min Brightness ≈ Max Brightness (difference ≤ 0.3)

Why:

* Exposure pumping destroys temporal stability.
* Sharpening + exposure variance = shimmer.

---

## Core controls that matter (and why)

### 1) Global Contrast (small, deliberate)

**Recommended range**

* `+0.05 → +0.15`

What it does:

* Increases separation between midtones.
* Helps TSR and sharpening recover perceived detail.

Why not more:

* High contrast exaggerates aliasing and specular noise.
* Creates halos when sharpened.

Rule:

* If you feel tempted to go above +0.15, fix lighting instead.

---

### 2) Global Gamma (often better than contrast)

**Recommended**

* Slightly below default (e.g., `0.95–0.98`)

What it does:

* Redistributes luminance without edge amplification.
* Often cleaner than contrast boosts.

Use when:

* Image feels flat but contrast increase causes artifacts.

---

### 3) Saturation (less than you think)

**Recommended**

* `0.95–1.05`

Why lower saturation often looks sharper:

* High saturation reduces luminance contrast between colors.
* Temporal filters operate primarily on luminance.
* Over-saturation hides detail and increases color crawl.

Rule:

* Increase saturation *after* contrast/gamma, not before.

---

### 4) Shadows (lift, don’t crush)

**Recommended**

* Shadows: slightly lifted
* Avoid deep black rolloff

Why:

* Crushed blacks:

  * kill texture detail,
  * produce temporal noise,
  * amplify sharpening artifacts.

Test:

* If dark foliage “boils” during motion, shadows are too crushed.

---

### 5) Highlights (compress gently)

**Recommended**

* Slight highlight compression
* Avoid hard clipping

Why:

* Specular highlights are the #1 cause of shimmer with TSR + sharpen.
* Compressed highlights:

  * reduce temporal instability,
  * still read as bright.

Rule:

* Bright ≠ clipped.

---

## Film settings (use sparingly)

If Film controls are enabled:

* Toe: gentle (prevents crushed shadows)
* Shoulder: soft (prevents specular clipping)
* Black Clip / White Clip: **do not touch unless necessary**

These act globally and can destabilize sharpening if misused.

---

## What *not* to do (common mistakes)

* Heavy LUTs early in development
* High contrast + high sharpen together
* Saturation-first grading
* Color grading to “fix” bad lighting
* Grading while Auto Exposure is unstable

---

## Interaction with TSR + Tonemapper Sharpen

Color grading directly affects how well sharpening behaves.

Bad grading → sharpening causes:

* halos,
* sparkle,
* crawling edges.

Good grading → sharpening produces:

* clean edges,
* stable detail,
* perceived resolution gain.

That’s why Recommendation 5 matters as much as AA choice.

---

## Practical “clean gameplay” baseline

As a starting point:

* Exposure: Manual or tightly clamped
* Contrast: +0.1
* Gamma: ~0.97
* Saturation: 1.0
* Shadows: slightly lifted
* Highlights: gently compressed
* LUTs: none

Then tune sharpen (not the other way around).

---

## Mental model (useful heuristic)

> **Lighting defines reality.
> Color grading preserves it.
> Sharpening only reveals what survives both.**

If you want, next we can:

* Build a **numerical PPV preset** you can copy verbatim
* Explain **why LUTs fight TSR**
* Or tune grading specifically for **foliage-heavy scenes** or **HUD-dominant games**

**Q: Will these settings ship with the game?**

**A:** Yes, if they are set in assets/config that are packaged, they ship; if they are editor-only overrides, they do not.

What ships

1. Post Process Volume settings

* Any Post Process Volume (PPV) actor placed in a cooked map ships with that map.
* PPV settings applied through:

  * Unbound (Infinite Extent) volumes
  * Bounded volumes (when camera is inside)
  * Blends by Priority/Blend Weight
* Camera component Post Process Settings (on player camera) also ship.

2. Config and project settings

* `DefaultEngine.ini`, `DefaultGame.ini`, `DefaultScalability.ini`, `DeviceProfiles.ini` ship (cooked into the build and/or staged alongside).
* Console variables (cvars) you set via:

  * `Engine/Config/ConsoleVariables.ini`
  * `[SystemSettings]` sections in `.ini`
  * Device Profiles
  * C++/Blueprint at runtime (e.g., Execute Console Command)
    will apply in packaged builds (subject to platform restrictions).

3. Quality selection logic

* Your game can ship with scalability presets and select them at runtime (auto-detect or user menu). Those choices ship.

What does not ship (common confusion)

* Editor viewport settings, PIE-specific quality tweaks, and editor “preview rendering level” overrides.
* Anything you only typed into the console during PIE and never persisted into config/Blueprint/C++.

Notes on “shipping build” restrictions

* Many cvars are still readable in Shipping, but interactive console access is often disabled.
* Some debug/show flags are compiled out.
* If you need a setting guaranteed in Shipping, set it in:

  * PPV / camera
  * `.ini` (project config)
  * Device Profile
  * Code/Blueprint at startup

Practical way to guarantee it ships

* Visual look settings (grading, DOF, motion blur, AA method): set in an Unbound PPV or camera.
* Performance/quality knobs (screen percentage, shadow quality, Lumen quality): set in Scalability + Device Profiles, then apply on boot.

If you tell me your Unreal Engine version and whether you want a fixed look or user-adjustable graphics menu, I can recommend the most robust place to store each setting.
