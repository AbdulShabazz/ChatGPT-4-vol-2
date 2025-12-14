## OpenAI ChatGPT v1.2025.330 Enable.One.File.Per.Actor.in.Unreal.Engine.md

**Q: How do I enable / enforce one file per actor in my Unreal Engine workflows?**

**A:** Enable it per-level via **World Settings → Use External Actors**, and batch-convert existing maps with `ConvertLevelsToExternalActorsCommandlet`.

Enable (single level; non–World Partition)

1. Open the map.
2. `Window → World Settings`
3. In **World** section, check **Use External Actors**.
4. When prompted, confirm conversion, then save the level. ([Epic Games Developers][1])

Notes

* If the level uses **World Partition**, **One File Per Actor (OFPA)** is enabled by default. ([Epic Games Developers][2])
* OFPA is **Editor-only**; when cooked, actor data is embedded back into the level package. ([Epic Games Developers][1])

Batch convert and “enforce” across many levels (including sublevels)
Run Epic’s conversion commandlet (example pattern below). It can convert a level and optionally all its sublevels. ([Epic Games Developers][1])

```bat
REM From Engine\Binaries\Win64
UnrealEditor.exe "C:\Path\Project\YourProject.uproject" ^
  -run="ConvertLevelsToExternalActorsCommandlet" ^
  -nosourcecontrol ^
  -convertsublevels ^
  "/Game/Maps/YourMapPath/YourMapName"
```

Workflow enforcement (practical options)

* **Template rule:** only create new worlds from templates that have **World Partition** (therefore OFPA) enabled by default, or immediately enable **Use External Actors** on any newly created non-partitioned map. ([Epic Games Developers][2])
* **Repository rule:** do **not** ignore `__ExternalActors__` (and related external-data folders); they are required for the map to load correctly under OFPA. (The exact folder names/layout are a byproduct of OFPA.) ([Epic Games Developers][1])
* **CI gate:** run the commandlet on candidate maps in Continuous Integration (CI) and fail the build if conversion produces changes (meaning someone introduced/committed a non-OFPA map).

Verification checklist

* After enabling/conversion, the project should gain/update external actor files associated with the map (commonly under a generated external-actors directory), and editing individual actors should typically touch those external files rather than rewriting the entire `.umap`. ([Epic Games Developers][1])
```
[1]: https://dev.epicgames.com/documentation/en-us/unreal-engine/one-file-per-actor-in-unreal-engine "One File Per Actor in Unreal Engine | Unreal Engine 5.7 Documentation | Epic Developer Community"
[2]: https://dev.epicgames.com/documentation/en-us/unreal-engine/world-partition-in-unreal-engine?utm_source=chatgpt.com "World Partition in Unreal Engine"
```
**Q: How to enable world partition?**

**A:** World Partition (WP) is enabled per level; you do not “toggle it on” for an existing `.umap` without converting that level.

New level (recommended)

1. `File → New Level`
2. Select **Open World** map type
3. Create the map (WP is enabled by default in this template). ([Epic Games Developers][1])

New project

* Create a new project from a **Games** template; many have WP enabled by default. ([Epic Games Developers][1])

Existing level (convert)
Editor conversion:

* Open the level, then `Tools → Convert Level…` (World Partition conversion). ([Epic Games Developers][1])

Command line conversion (Windows):

```bat
REM Run from Engine\Binaries\Win64 (path containing UnrealEditor.exe)
UnrealEditor.exe "C:\Path\Project\YourProject.uproject" ^
  -run=WorldPartitionConvertCommandlet ^
  "C:\Path\Project\Content\Maps\YourMap.umap" ^
  -AllowCommandletRendering
```

Epic’s doc shows the required structure and example invocation. ([Epic Games Developers][1])

Streaming toggle (separate from “WP enabled”)

* Grid-cell streaming can be enabled/disabled via **World Settings → Enable Streaming** after WP is in use. ([Epic Games Developers][1])

[1]: https://dev.epicgames.com/documentation/en-us/unreal-engine/world-partition-in-unreal-engine "World Partition in Unreal Engine | Unreal Engine 5.7 Documentation | Epic Developer Community"

**Q: Where is World Settings? What is a world partition minimap?**

**A:** ## Where is World Settings?

* Main menu: **Window → World Settings**. ([Epic Games Developers][1])
* Default docking location: typically **next to the Details panel** in the Level Editor UI. ([Epic Games Developers][1])
* If it is “missing” (tab closed/off-screen), reset layout: **Window → Load Layout → Default Editor Layout**. ([Epic Developer Community Forums][2])

## What is a World Partition minimap?

A **World Partition (WP) minimap** is an **editor-only** minimap image/asset used to **preview and navigate the world inside the World Partition window** (select regions / grid cells, load/unload regions, etc.). ([Epic Games Developers][3])

Related practical points:

* Open the WP window: **Window → World Partition → World Partition Editor**. ([Epic Games Developers][3])
* Generate/update the minimap: **Build menu → World Partition → Build Minimap** (or via a builder commandlet). ([Epic Games Developers][3])
* If the minimap does not appear after building, Epic notes enabling **Virtual Textures (VT)** support may be required. ([Epic Games Developers][3])
```
[1]: https://dev.epicgames.com/documentation/en-us/unreal-engine/world-settings-in-unreal-engine "World Settings in Unreal Engine | Unreal Engine 5.7 Documentation | Epic Developer Community"
[2]: https://forums.unrealengine.com/t/world-settings-not-showing-up/743435?utm_source=chatgpt.com "World settings not showing up."
[3]: https://dev.epicgames.com/documentation/en-us/unreal-engine/world-partition-in-unreal-engine?utm_source=chatgpt.com "World Partition in Unreal Engine"
```
**Q: How do I debug world partition?**

**A:** ## World Partition debugging workflow

### 1) Verify editor-side state (cells exist, regions load)

* Open **World Partition Editor**: `Window → World Partition → World Partition Editor`. ([Epic Games Developers][1])
* Manually **Load Region**: drag-select an area → right-click → **Load Region from Selection**. ([Epic Games Developers][1])
* If navigation is painful, **Build Minimap**: `Build → World Partition → Build Minimap` (or run the minimap builder commandlet). ([Epic Games Developers][1])

  * If minimap does not appear, enable **Virtual Texture support** in Project Settings. ([Epic Games Developers][1])

Useful editor shortcuts in the WP window:

* `Ctrl + Double Click`: load region around cursor
* `Shift + Double Click`: start Play-In-Editor (PIE) at cursor ([Epic Games Developers][1])

### 2) Runtime visualization (PIE / Standalone)

Open the in-game console and toggle overlays:

* `wp.Runtime.ToggleDrawRuntimeHash2D` / `wp.Runtime.ToggleDrawRuntimeHash3D` (cell streaming state + runtime hash) ([Epic Games Developers][1])
* `wp.Runtime.ToggleDrawStreamingSources` (streaming sources visualization) ([Epic Games Developers][2])
* `wp.Runtime.ToggleDrawRuntimeCellsDetails` (per-cell details) ([Epic Games Developers][2])
* `wp.Runtime.ToggleDrawStreamingPerfs` (streaming perf overlay) ([Epic Games Developers][2])
* `wp.Runtime.ToggleDrawLegends` (legend overlay) ([Epic Games Developers][2])

Narrow what you see:

* `wp.Runtime.DebugFilterByCellName <partial>` ([Epic Games Developers][2])
* `wp.Runtime.DebugFilterByDataLayer <labels>` ([Epic Games Developers][2])
* `wp.Runtime.DebugFilterByRuntimeHashGridName <grid>` ([Epic Games Developers][2])
* `wp.Runtime.DebugFilterByStreamingStatus <status>` ([Epic Games Developers][2])

### 3) Dump ground truth to the Output Log (fast triage)

* `wp.Runtime.DumpStreamingSources` (proves whether any streaming source is active) ([Epic Games Developers][2])
* `wp.Runtime.DumpWorldPartitions` ([Epic Games Developers][2])
* `wp.Runtime.DumpDataLayers` ([Epic Games Developers][2])

If **no streaming sources** are active, streaming will not progress. Use the dump + `ToggleDrawStreamingSources` to confirm. ([Epic Games Developers][2])

### 4) Data Layer (DL) issues (cells loaded but content “missing”)

* Visualize: `wp.Runtime.ToggleDrawDataLayers` / `wp.Runtime.ToggleDrawDataLayersLoadTime` ([Epic Games Developers][2])
* Force state: `wp.Runtime.SetDataLayerRuntimeState <Unloaded|Loaded|Activated> <DataLayerNames...>` ([Epic Games Developers][2])
* Quick toggle: `wp.Runtime.ToggleDataLayerActivation <DataLayerNames...>` ([Epic Games Developers][2])

### 5) Increase logging when behavior is inconsistent

* `wp.Runtime.SetLogWorldPartitionVerbosity` to raise World Partition log verbosity. ([Epic Games Developers][2])

### 6) Editor-side diagnostics (bad actor descriptors / generation)

Run these from the console / Output Log in-editor:

* `wp.Editor.DumpActorDesc <Actor>` (single) ([Epic Games Developers][2])
* `wp.Editor.DumpActorDescs` (CSV) ([Epic Games Developers][2])
* `wp.Editor.DumpStreamingGenerationLog` (generation log) ([Epic Games Developers][2])
* `wp.Editor.HLOD.DumpStats` (CSV stats) ([Epic Games Developers][2])

### 7) Controlled experiments (is it streaming logic or content?)

* Temporarily override loading range: `wp.Runtime.OverrideRuntimeSpatialHashLoadingRange -grid=<index> -range=<value>` ([Epic Games Developers][2])
* Toggle HLOD loading/rendering: `wp.Runtime.HLOD <0|1>` ([Epic Games Developers][2])

### 8) Common “gotcha”: Level Blueprint hard references

Actors referenced in a **Level Blueprint** can be marked **Always Loaded**; prefer Blueprint Classes for runtime references. ([Epic Games Developers][1])

If you describe the symptom (e.g., “no cells stream in PIE”, “cells stream but actors missing”, “client falls through landscape”, “teleport leaves world unloaded”), the shortest command sequence to capture evidence is:
`wp.Runtime.DumpStreamingSources`, `wp.Runtime.ToggleDrawRuntimeHash2D`, `wp.Runtime.ToggleDrawStreamingSources`, `wp.Runtime.DumpDataLayers`, `wp.Runtime.SetLogWorldPartitionVerbosity`. ([Epic Games Developers][2])
```
[1]: https://dev.epicgames.com/documentation/en-us/unreal-engine/world-partition-in-unreal-engine "World Partition in Unreal Engine | Unreal Engine 5.7 Documentation | Epic Developer Community"
[2]: https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-console-commands-reference "Unreal Engine Console Commands Reference | Unreal Engine 5.7 Documentation | Epic Developer Community"
```
