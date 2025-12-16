## OpenAI ChatGPT v1.2025.336 HUD.UI.Blueprint.Setup.in.UE.md

**Q: In Unreal Engine Blueprints, help me build this HUD UI! Boilerplate: Niagara is desired for stylized meters with slide-in animations; composite fonts which are system independent are in use: DA_Typography, has Regular, Medium, and Bold presets (roboto royalty-free TTF Libraries); notice the horizontal compass across the top edge which also supports distance to waypoints; notice the minimap and its likely functionality; create suitable names for all assets!!**

**A:** Result: implement as a root Unreal Motion Graphics (UMG) widget composed of three sub-widgets (Compass, Stats, Minimap) driven by one HUD controller (Player Controller + HUD data component). Use UMG animations for slide-in, and Niagara only via a UI-compatible path (plugin or Render Target (RT) capture), because stock Niagara does not render directly inside UMG.

Asset set (suggested names)

* Data

  * `DA_Typography` (given; presets: Regular/Medium/Bold using Roboto Composite Font)
  * `DA_HUDTheme_SciFi01` (colors, opacities, sizes, padding, meter widths)
  * `DT_CompassTicks_SciFi01` (optional; tick layout tuning)
  * `ST_WaypointHUDEntry` (Struct: Actor, DisplayName, Color, bIsObjective, Icon)
* Widgets

  * `WBP_HUD_Root`
  * `WBP_CompassBar`
  * `WBP_CompassWaypointMarker`
  * `WBP_StatStack` (container for HP/MP/XP)
  * `WBP_StatMeter_Niagara` (one meter row: label + stylized bar)
  * `WBP_Minimap`
  * `WBP_MinimapBlip`
* Blueprints / Components

  * `BP_HUDPresenterComponent` (Actor Component on Player Controller; centralizes HUD state)
  * `BPI_HUDDataProvider` (Blueprint Interface for health/mana/xp/waypoints)
  * `BP_MinimapCaptureRig` (Actor with SceneCaptureComponent2D)
* Materials / Textures

  * `M_Minimap_CircleMask` + `MI_Minimap_CircleMask_SciFi01`
  * `M_CompassTape_UVScroll` + `MI_CompassTape_SciFi01` (optional material-based tape)
  * `T_UI_HUDLine_01`, `T_UI_Tick_01`, `T_UI_Arrow_Player_01`, `T_UI_Blips_Atlas_01`
* Render targets

  * `RT_Minimap_512` (or 1024 if needed)
* Niagara (two viable paths; pick one)

  * Path A (preferred): Niagara UI Renderer plugin

    * `NS_UI_MeterFill_SciFi01` (User params: Fill01, Color, Pulse, NoisePhase)
  * Path B (stock engine): world Niagara + SceneCapture2D to RT, then show RT in UMG

    * `NS_World_MeterFill_SciFi01` + `BP_MeterCaptureRig` + `RT_MeterFill_256`

UI hierarchy (WBP_HUD_Root)

* Canvas Panel

  * Top: `WBP_CompassBar` anchored top-center, full width, ~48–64 px height
  * Right: `WBP_StatStack` anchored top-right, three rows (HP, MP, XP)
  * Bottom-right: `WBP_Minimap` anchored bottom-right, circular masked image + border + player arrow

1. Compass bar (WBP_CompassBar)
   Visual behavior

* Thin horizontal rule across the top, with tick marks and letters N/E/S/W.
* Center caret/triangle at screen center.
* Waypoint marker(s) slide along the tape with distance readout centered under the caret (example “150m”).

Recommended implementation (UMG-native, no material required)

* Build a “tape” inside a `SizeBox` (fixed height) + `CanvasPanel`:

  * `Overlay`

    * `HorizontalBox` (the tape content; ticks + labels repeated)
    * `Image_CenterCaret`
    * `CanvasPanel_Waypoints` (markers positioned by computed X)
* Tape content strategy

  * Pre-create 360° worth of tick widgets is heavy; instead create a repeating segment:

    * Segment covers 90° (N→E) containing ticks every 5°, major ticks every 15°, labels at N/NE/E etc.
    * Duplicate segments 4x (N→E, E→S, S→W, W→N) to cover 360°.
  * At runtime, shift the whole tape left/right via Render Transform Translation X.

Runtime math (tick scroll)

* Let:

  * `YawDeg = NormalizeAxis(PlayerYawWorld)` in degrees (0–360, where 0 = North)
  * `PixelsPerDegree` from theme (e.g., 6.0 px/deg; tune)
  * `TapeOffsetX = -YawDeg * PixelsPerDegree`
* Apply to tape container: `SetRenderTranslation( (TapeOffsetX, 0) )`
* To keep the center aligned, also add half-width correction once (store in `OnInitialized`):

  * `TapeOffsetX += TapeCenteringBiasPx`

Waypoint marker placement

* For each waypoint Actor:

  * `Dir = WaypointLocation - PlayerLocation`
  * `BearingDeg = NormalizeAxis( atan2(Dir.Y, Dir.X) in degrees )` (convert to world bearing where 0 = +X/East; adjust to your project’s North definition)
  * Convert to “compass degrees relative to North” consistently with your yaw source.
  * `DeltaDeg = FindDeltaAngleDegrees(YawDeg, BearingDeg)`
  * `MarkerX = CenterX + DeltaDeg * PixelsPerDegree`
  * Clamp to bar bounds if desired (hide when off-bar, or pin to edges).
* Distance readout under caret

  * Choose “active waypoint” (nearest objective or selected).
  * `DistM = round( |Dir| / 100 )` (Unreal units to meters)
  * Set text: `"{DistM}m"` using `DA_Typography.Medium`.

Blueprint events

* `WBP_CompassBar::Tick` (or timer at 30 Hz via `BP_HUDPresenterComponent`):

  * Pull `PlayerYaw`, `Waypoints[]`, `ActiveWaypoint`.
  * Update tape translation and marker positions.
* Prefer timer-driven updates over per-frame if you want deterministic cost.

2. Stat meters stack (WBP_StatStack + WBP_StatMeter_Niagara)
   Visual behavior

* Three labeled meters at top-right: HP (red), MP (blue), XP (light blue).
* Slide-in on HUD show.
* Stylized fill (glow/noise) preferably Niagara-driven.

Slide-in (UMG animation)

* In `WBP_StatStack`, create `Anim_Intro_SlideInRight`:

  * Initial: RenderTranslation X = +200, Opacity = 0
  * End: X = 0, Opacity = 1
* Play in `OnInitialized` or when HUD is shown.

Meter widget layout (WBP_StatMeter_Niagara)

* `Overlay`

  * `Text_Label` (HP/MP/XP) using `DA_Typography.Bold`
  * `Image_Backplate` (thin line)
  * `NiagaraOrFillImage` (the animated fill layer)
* Input API (Expose on Spawn)

  * `MeterName` (Name)
  * `LabelText` (Text)
  * `Color` (LinearColor)
  * `Percent` (0..1)

Niagara integration (choose one)

* Path A: Niagara UI Renderer plugin

  * Add `NiagaraSystemWidget` (or plugin’s widget) to `WBP_StatMeter_Niagara`.
  * Set User Params each update:

    * `Fill01 = Percent`
    * `Tint = Color`
* Path B: stock engine workaround

  * Spawn a world-space Niagara system attached to a hidden capture rig, render it with a small `SceneCaptureComponent2D` into `RT_MeterFill_256`, then display that RT in the widget as an `Image` brush.
  * This is heavier; only do it if Path A is not allowed.

Blueprint update flow

* `BP_HUDPresenterComponent` owns normalized values:

  * `HP01`, `MP01`, `XP01`
* When values change (event-driven):

  * Call `WBP_StatStack::SetMeters(HP01, MP01, XP01)`
  * Each meter calls `SetPercent` and updates Niagara/material params.

3. Minimap (WBP_Minimap + BP_MinimapCaptureRig)
   Core functionality

* Top-down capture to `RT_Minimap_512`.
* Circular mask + border.
* Player arrow centered.
* Blips for points of interest and waypoints.
* Optional rotation: north-up or player-up.

Capture rig (BP_MinimapCaptureRig)

* Components

  * `SceneCaptureComponent2D_Minimap`
  * `SpringArm` (optional) to maintain altitude
* Settings

  * Projection: Orthographic
  * Ortho Width: tune to show desired meters (e.g., 2000–4000 uu)
  * Capture Source: `Final Color (LDR)` or custom depth-based stylization
  * Texture Target: `RT_Minimap_512`
  * ShowOnly list or custom rendering layer for clean minimap (recommended)

Widget (WBP_Minimap)

* `Image_MinimapRT` (brush from `RT_Minimap_512`)
* Apply `MI_Minimap_CircleMask_SciFi01` to mask it (material does circle alpha + subtle rim)
* `Image_PlayerArrow` centered; rotate if north-up
* `CanvasPanel_Blips` for blip widgets, positioned in minimap space

Blip positioning math (north-up example)

* Let:

  * `WorldDelta = TargetLocation - PlayerLocation` (X,Y only)
  * `Scale = PixelsPerUU` derived from OrthoWidth and widget radius
  * If north-up: rotate delta by `-PlayerYaw` so the map stays fixed and the arrow rotates, or keep arrow fixed and rotate the map; pick one.
* Convert to widget space:

  * `Local = Rotate2D(WorldDelta, -PlayerYawDeg)` (if player-up map)
  * `BlipPosPx = Local * Scale`
  * Clamp to circle radius minus padding; optionally draw edge arrow when clamped.

Waypoint cohesion

* Reuse the same waypoint list used by the compass.
* The “active waypoint” drives:

  * Compass distance text
  * A distinct minimap blip style (larger, pulsing)

4. Wiring and ownership (Blueprint responsibilities)

* Player Controller (PC)

  * Owns `BP_HUDPresenterComponent`
  * Creates `WBP_HUD_Root` on BeginPlay and adds to viewport
* `BP_HUDPresenterComponent`

  * Queries your gameplay systems via `BPI_HUDDataProvider` (health, mana, xp, waypoints)
  * Emits event-driven updates to widgets (preferred)
  * Runs a 30 Hz timer for compass/minimap tracking if you want bounded cost

Minimum public APIs (recommended)

* `BPI_HUDDataProvider`

  * `GetHealth01()`, `GetMana01()`, `GetXP01()`
  * `GetWaypoints(out Array<ST_WaypointHUDEntry>)`
  * `GetActiveWaypoint(out ST_WaypointHUDEntry)`
* `WBP_HUD_Root`

  * `BindPresenter(BP_HUDPresenterComponent)`
* `WBP_CompassBar`

  * `SetPlayerYaw(float)`, `SetWaypoints(Array)`, `SetActiveWaypoint(Entry)`
* `WBP_StatStack`

  * `SetHP01(float)`, `SetMP01(float)`, `SetXP01(float)`
* `WBP_Minimap`

  * `SetCaptureTexture(RT)`, `SetBlips(Array)`

Notes on Niagara in UMG

* Stock Unreal Engine: Niagara renders to the scene, not directly to UMG. If Niagara must be the meter fill layer inside UI, use a Niagara-to-UMG solution (UI renderer plugin) or capture Niagara to a Render Target and display it. If you accept “Niagara-like” visuals, a UMG material with noise/pulse parameters is the simplest and cheapest.

If you want, specify: north-up vs player-up minimap, number of simultaneous waypoints, and whether the compass tape should be material-driven (single quad UV scroll) or widget-driven (repeated tick widgets).
